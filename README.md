# Ishan Srivastava: Portfolio

Source for my portfolio site, built with Jekyll and the Minimal theme and published with GitHub Pages at <https://ishan-phys.github.io>.

The site presents my experience, education and selected projects in applied AI, data engineering and machine learning. Each project has a short write-up covering the problem and motivation, how it was solved, and a summary of the result, with a link to its repository.

## Structure

| Path | Purpose |
| --- | --- |
| `index.md` | Portfolio content: all project write-ups |
| `_config.yml` | Site title, header text and links, theme |
| `_layouts/default.html` | Page layout |
| `images/` | Project figures and screenshots |
| `pdf/` | Résumé and Master's thesis report |

## Run locally

```sh
bundle init
bundle add jekyll github-pages
bundle exec jekyll serve
```

Then open <http://127.0.0.1:4000>.
