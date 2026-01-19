# go-powers

A Claude Code skill marketplace for idiomatic Go programming.

## Overview

This skill transforms Claude into a strict, idiomatic Go programmer who writes and reviews
code as if Rob Pike, Dave Cheney, and the Google Go team were watching.

## Installation

### Via Marketplace (Recommended)

Add the marketplace and install the plugin:

```sh
/plugin marketplace add jamesainslie/go-powers
/plugin install go-powers@go-powers
```

Then use the skill:

```sh
/skill go-powers:idiomatic-go
```

### Local Development

Clone and add as local marketplace:

```bash
git clone https://github.com/jamesainslie/go-powers.git
cd go-powers
```

Then in Claude Code:

```sh
/plugin marketplace add .
/plugin install go-powers@go-powers
```

## What It Enforces

### Go Proverbs (All 19)

- "Clear is better than clever"
- "Errors are values"
- "The bigger the interface, the weaker the abstraction"
- "Don't panic"
- And 15 more...

### Domain-Specific Rules

- **Error Handling**: Always wrap with context, never ignore
- **Interface Design**: Small interfaces, consumer-defined, accept interfaces return structs
- **Concurrency**: Every goroutine has a known lifecycle, context propagation
- **Naming**: No Get prefix, no stutter, no util packages
- **Testing**: Table-driven, subtests, t.Helper()

### Red Flags & Anti-Patterns

- 30+ red flags to catch
- Code examples of what NOT to do
- Rationalization counters for common excuses

## Sources

- [Go Proverbs](https://go-proverbs.github.io/) - Rob Pike
- [The Zen of Go](https://dave.cheney.net/2020/02/23/the-zen-of-go) - Dave Cheney
- [Google Go Style Guide](https://google.github.io/styleguide/go/)
- [Uber Go Style Guide](https://github.com/uber-go/guide)
- [Effective Go](https://go.dev/doc/effective_go)

## Contributing

1. Fork this repo
2. Make changes to `idiomatic-go/SKILL.md`
3. Test locally by copying to `~/.config/superpowers/skills/`
4. Submit PR

## License

MIT
