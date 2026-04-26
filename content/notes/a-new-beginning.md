---
{
    "title": "A New Beginning",
    "date": "2026-03-27",
    "description": "Why I started this blog, and a quick test of the engine.",
    "keywords": "A new Beginning, a new start blog, Raditya Mahatma Ghosi, SiputBiru",
    "pinned": true,
    "draft": false
}
---

Welcome to my new digital notebook.

I set this space up because I needed a dedicated place to share my thoughts, but more importantly, I needed a place to leave notes for myself. When you are deep in the trenches of systems programming, building graphics libraries, or debugging C code, it is incredibly easy to lose track of the bigger picture.

This blog is my anchor. It is a repository of my past ideas and technical explorations that I can re-read whenever I need a reminder. It is here to make sure my original motivations stay clear and don't get forgotten in the daily grind.

I am hosting these notes on a custom Static Site Generator I built entirely from scratch in Rust (I will write a dedicated post about *how* and *why* I built that later). But since this is the maiden voyage[^1], I need to make sure the engine can actually render my notes properly.

Let's run it through some tests.

## Things that need to work perfectly

First, standard text formatting like **Bold text** and *italic text* parsing, along with custom JSON frontmatter extraction.

Next, routing images correctly from the public directory:

Image from the internet:
![cat-1](https://placecats.com/300/200)

Image from local storage:
![cat-2]({{ root_dir }}images/cat.jpeg)

I get all of the sample cat image from [placecats.com](https://placecats.com)

### CodeBlock

It also needs to handle inline code like `let x = 10;` flawlessly without breaking the surrounding paragraph.

full codeblock things:

```Typescript
console.log("Hello From Cangkang!");
```

### Callout

> [!NOTE]  
> Highlights information that users should take into account, even when skimming.

> [!TIP]
> Optional information to help a user be more successful.

> [!IMPORTANT]  
> Crucial information necessary for users to succeed.

> [!WARNING]  
> Critical content demanding immediate user attention due to potential risks.

> [!CAUTION]
> Negative potential consequences of an action.

### Numbered List

nested bullet item

1. Main item
    * Nested bullet item
    * Another nested bullet item

nested numbered item
2. Second main item
    1. Nested numbered item
    2. Another nested numbered item

### Footnotes & Tables

Sometimes my notes will need extra context, so it has to handle secret footnotes attached to paragraphs[^2]. You actually have seen in the definition of Maiden Voyage.

Table:

| Feature | Supported | Alignment |
| :--- | :---: | ---: |
| **Bold Text** | Yes | Center |
| *Callouts* | Yes | Right |
| Tables | Yes | *Magic* |

[^1]: The very first journey or trip made by a ship, boat, or sometimes an aircraft after its construction and testing. It traditionally represents a new beginning. [source](https://dictionary.cambridge.org/dictionary/english/maiden-voyage)

[^2]: This is the footnote text! Cangkang grabs this text and turns it into a beautiful, interactive margin note so it won't clutter the main paragraph.

### Testing Code Dropdowns

This is a regular code block:

```rust
fn main() {
    println!("Hello, world!");
}
```

This is a dropdown code block with a title:

+++rust [Main Function]
fn main() {
    println!("Hello from the dropdown!");
}
+++

This is a dropdown code block without a title:

+++javascript
console.log("No title here");
+++

This is a dropdown code block with multiple +++ inside (it should handle it):

+++++text [Nested Plus Test]
Some text
++++
More pluses
++++
Back to three
+++++
