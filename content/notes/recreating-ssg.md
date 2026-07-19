---
{
    "title": "Recreating the SSG from Scratch",
    "date": "2026-03-28",
    "description": "Why I built my own Static Site Generator instead of using modern frameworks.",
    "keywords": "Recreational Programming, Rust, custom SSG, Raditya Mahatma Ghosi, SiputBiru",
    "draft": yes
}
---

## Motivation

First things first, we need to talk about why I am even doing this.

Before we jump into *how* I built [Cangkang](https://github.com/SiputBiru/cangkang)[^1], I originally planned to create this blog using something like Astro. But recently, I was watching a video by Gingerbill[^2], the creator of the Odin Programming Language[^3], where he created an SSG completely by himself. It made me stop and wonder: *"Why don't I just do it myself?"*

Usually, when someone want to spin up a blog, we reach for the usual suspects—React, Astro, or Vite SSG. But honestly, I think that is just too boring. I wanted to build this from the ground up.

## How

With the motivation out of the way, let's jump right into how I actually built this thing.

### Project Plan

Everything starts with a plan. I wanted to build this SSG engine entirely from scratch with zero external dependencies, written purely in Rust. Why Rust? I had heard that Rust has some really powerful string manipulation capabilities(compared to C ofc), so I just wanted to put it to the test.

### Project Structure

![cangkang-project-structure]({{ root_dir }}images/recreating-ssg/project-structure-02.svg)

i created it with this structure is to make it as simple as possible, but still having proper (but not production ready) ssg that i can use easily and fix it easily.

the content and public is just the "asset" that the engine will read, the output will be places inside dist directory then web server will serve it as is without any other js or ssr stuff. this makes the engine is really just about reading the markdown -> lexer do tokenization -> parser -> html.

### Technical Flow

The technical flow is orchestrated by the compiler[^4], which acts as the central logic engine. The sequence for each file and the site as a whole is:

![cangkang-project-structure]({{ root_dir }}images/recreating-ssg/technical-flow-svg.svg)

  1. **FS (Input)**: The compiler uses fs to read raw Markdown files and copy the public/ assets to dist/.
  2. **Frontmatter**: The compiler extracts JSON-like metadata (title, date, etc.) from the top of the Markdown file.
  3. **Lexer**: The raw Markdown text is passed to the lexer to be converted into a stream of tokens.
  4. **Parser**: The parser consumes those tokens and constructs an Abstract Syntax Tree (AST) representing the document structure.
  5. **HTML**: The renderer takes the AST and converts it into a rendered HTML string (the page body).
  6. **Compiler (Template Injection)**: The compiler takes that rendered HTML and performs string substitution on the HTML templates ({{ content }}, {{ title }}, etc.).
  7. **SEO**: After all pages are processed, the compiler passes the collected metadata to the seo module to generate site-wide assets like the sitemap.
  8. **FS (Output)**: The final, fully-formed HTML files are written to the dist/ directory

### Main Setup

the way i will use it much more like this: run the program, the program will read the dir i want to read then convert it to html with template.

the `main.rs` will just looks like this:

```rust
fn main() {
  if let Err(e) = compiler::build_site() {
    log_error;
    eprintln;
    std::process::exit(1);
  }
}
```

really simple right?

### Handling Error with style

To keep our code clean, we define a `CangkangError` enum. This groups all possible failures—from missing files to bad Markdown syntax—into one type-safe structure.

the `CangkangError` enum will just like this:

```rust
#[derive(Debug)]
pub enum CangkangError {
    // IO Error with option context
    Io(String, io::Error),

    // Markdown parsing errors
    Parse { message: String, line: usize },

    // Frontmatter syntax errors
    Frontmatter(String),

    // HTML Template errors
    Template(String),
}
```

To make these errors helpful for users, we can implement `fmt::Display`. This allows us to customize the output, adding specific context (like a file path or a line number) only when it's available.

```rust
impl fmt::Display for CangkangError {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        match self {
            // Compressed IO logic using a simple prefix
            Self::Io(ctx, err) if ctx.is_empty() => write!(f, "IO Error: {err}"),
            Self::Io(ctx, err) => write!(f, "IO Error at '{ctx}': {err}"),
            
            // Inline the rest for brevity
            Self::Parse { message, line } => write!(f, "Parse Error (line {line}): {message}"),
            Self::Frontmatter(msg) => write!(f, "Frontmatter Error: {msg}"),
            Self::Template(msg) => write!(f, "Template Error: {msg}"),
        }
    }
}
```

Finally, we use an Extension Trait called `IoContext`. This is a bit of Rust "magic" that lets us add the `.with_ctx()` method directly onto Rust's standard Result.

```rust
pub trait IoContext<T> {
    fn with_ctx<S: Into<String>>(self, ctx: S) -> Result<T, CangkangError>;
}

impl<T> IoContext<T> for Result<T, io::Error> {
    fn with_ctx<S: Into<String>>(self, ctx: S) -> Result<T, CangkangError> {
        self.map_err(|e| CangkangError::Io(ctx.into(), e))
    }
}
```

The result? Instead of writing complex match statements every time we open a file, we can simply chain our error handling:

before:

```rust
let data = fs::read_to_string("config.toml")
    .map_err(|e| CangkangError::Io("config.toml".to_string(), e))?;
```

after:

```rust
// example with read_to_string
let content = fs::read_to_string(path).with_ctx(path)?;
```

### FileSystem

Then we will start with the FileSystem. whats actually filesystem? this is just more like library that will be used as a file & folder maker.

There are several ways to read a file in Rust: loading the entire thing at once, processing it line-by-line (buffered), reading raw bytes, or even using memory mapping with crates like mmap[^5] for massive datasets.

Since Markdown files are typically small, we’ll stick to the most straightforward method: `fs::read_to_string`.

```rust
pub fn read_markdown_file<P: AsRef<Path>>(file_path: P) -> Result<String, CangkangError> {
    let path = file_path.as_ref();
    fs::read_to_string(path).with_ctx(path.to_string_lossy())
}
```

I used `path.to_string_lossy()` here for our error context. This is a safe choice because it gracefully handles non-UTF-8 characters by replacing them with the Unicode Replacement Character (U+FFFD)[^6], this ensuring our program never crashes just because of a weird filename.

Writing the processed HTML is just as simple. However, there is one extra step: we need to ensure the destination directory actually exists before we try to save the file.

```rust
pub fn write_html_file<P: AsRef<Path>>(file_path: P, content: &str) -> Result<(), CangkangError> {
    let path = file_path.as_ref();

    if let Some(parent) = path.parent() {
        fs::create_dir_all(parent).with_ctx(parent.to_string_lossy())?;
    }

    fs::write(path, content).with_ctx(path.to_string_lossy())
}
```

Finally, we need a way to copy assets (like CSS or images) from our source to our output folder. Since `std::fs` doesn't provide a recursive copy function out of the box, we’ll implement a simple one ourselves.

```rust
pub fn copy_dir_all(src: impl AsRef<Path>, dst: impl AsRef<Path>) -> Result<(), CangkangError> {
    let src = src.as_ref();
    let dst = dst.as_ref();

    if !src.exists() {
        return Ok(());
    }

    fs::create_dir_all(dst).with_ctx(dst.to_string_lossy())?;

    for entry in fs::read_dir(src).with_ctx(src.to_string_lossy())? {
        let entry = entry.with_ctx("directory entry")?;
        let file_type = entry.file_type().with_ctx("file type")?;
        let dst_path = dst.join(entry.file_name());

        if file_type.is_dir() {
            copy_dir_all(entry.path(), dst_path)?;
        } else {
            fs::copy(entry.path(), &dst_path).with_ctx(format!(
                "{} to {}",
                entry.path().display(),
                dst_path.display()
            ))?;
        }
    }

    Ok(())
}
```

This function scans a directory and, if it finds another folder inside, calls itself again. This "recursion" allows us to mirror even the most complex folder structures perfectly.

### Lexer (Lexical analysis)

If you ever wondering what is actually lexer, lexer is much more like a someone that will breakdown our markdown to chunk named Tokens. 

we can define our token with enum much more like this:
```rust
#[derive(Debug, PartialEq)]                     
pub enum Token {                                
    HeadingMarker(u8),  // # ## ### — carries   
the level count                                 
    Text(String),       // plain text           
    Newline,            // \n                   
    Asterisk,           // *                    
    BracketLeft,        // [                    
```

in my implementation the lexer also have little bit of functionality to read each char. the code is much more like this:
```rust
pub fn read_char(&mut self) {
    if self.read_position >= self.input.len() {
        self.ch = '\0';
    } else {
        self.ch = self.input[self.read_position];
    }
    self.position = self.read_position;
    self.read_position += 1;
}
```

and also there is next_token function that will actually use the read_char to check each char
```rust
pub fn next_token(&mut self) -> Token {
    match self.ch {
        '\n' => {
            self.read_char(); // Advance past the matched character for single-char tokens
            Token::Newline
        }
        '*' => {
            self.read_char();
            Token::Asterisk
        }
        '\0' => Token::Eof,
        '#' => {
            let mut level = 0;
            // Count how many '#' we have
            while self.ch == '#' {
                level += 1;
                self.read_char();
            }
            // Skip the trailing space if there is one
            if self.ch == ' ' {
                self.read_char();
            }
            Token::HeadingMarker(level) // Early return because we already advanced
        }
        '[' => {
            self.read_char();
            Token::BracketLeft
// etc
```
the next_token actually wil skip if there is space or something like that. then lexer will return the respective token that being associate with those char. it can be explained much more like this:
The lexer reads one character at a time. For each character, it asks a question:
- Is it #? → Count how many → emit HeadingMarker(n)
- Is it *? → emit Asterisk
- Is it [ or ] or ( or )? → emit the matching bracket token
- Is it \n? → emit Newline
- Is it anything else? → keep reading until we hit a special character → emit the whole chunk as Text


### Parser

After the Lexer breaking down the markdown file into tokens, parser will just interprete what each token mean. The parser will create AST[^7]. later the compiler and the html.rs will handle the conversion.


### Compiler

## Conclusion

[^1]: *Cangkang* is the Indonesian word for "shell". Since I go by SiputBiru (Blue Snail), it felt fitting that the engine hosting my notes would be my shell

[^2]: [Gingerbill "I Built My Own Static Site Generator With Odin in an Afternoon!"](https://www.youtube.com/watch?v=YvnTsiIFXeI)

[^3]: [Odin Programming Language](https://odin-lang.org/)

[^4]: not really compiler we know like gcc or javac but we can still called it [compiler](https://en.wikipedia.org/wiki/Compiler) right?

[^5]: i never actually use it but it looks cool tho. [mmap](https://github.com/TIVerse/mmap-rs)

[^6]: [Special Unicode Code](https://en.wikipedia.org/wiki/Specials_(Unicode_block))

[^7]: [Abstract Syntax Tree](https://en.wikipedia.org/wiki/Abstract_syntax_tree)
