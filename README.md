# Just my blog

## updating the submodules

```bash
git submodule update --remote --merge
```

## running the things

```bash
cargo run --manifest-path ssg/Cargo.toml --release
```

running with docker compose:

```bash
docker compose up --build -d
```
