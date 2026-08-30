# Mirror Design

A personal collection of reverse-engineered and reconstructed design systems, built for reference, learning, and backup.

## Design Systems

| Library | Source | Style |
|---------|--------|-------|
| [OpenAI](./.design_library/OpenAI/) | [openai.com](https://openai.com/zh-Hans-CN/) | Minimal white-canvas with single green accent |
| [Voith](./.design_library/Voith/) | [voith.com](https://www.voith.com/corp-en/about-us/markets-locations/china-cn.html) | Industrial marine-blue corporate engineering |
| [Endfield](./.design_library/Endfield/) | [endfield.hypergryph.com](https://endfield.hypergryph.com/) | Industrial post-apocalyptic dark UI with triple-accent (yellow/green/magenta) |

Each library includes:
- Token system (`colors_and_type.css` + `css.json`)
- Component contracts (`components/*.json`)
- Component previews (`preview/*.html`)
- Marketing UI Kit (`ui_kits/marketing/index.html`)
- Documentation (`README.md` + `SKILL.md`)

## Structure

```
mirror_design/
  .design_library/
    OpenAI/          # Minimal white + green
    Voith/           # Industrial blue + cyan
    Endfield/        # Dark + hazard yellow + neon green + magenta
```

## License

MIT
