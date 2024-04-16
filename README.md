# mptcp.dev website

This README is meant as a guide on the structure and particularities of this
website for those who want to contribute.

## The pages
- `index`, the landing page should contain basic context of this website and
  explanations on what MPTCP is. Also a section dedicated to links to other
  MPTCP related content.
- `setup` is meant as a guide for end users, helping them setup and use MPTCP on
  their system.
- `debugging` is meant as a source of solutions on how and what to use to debug
  MPTCP on a system.
- `implementation` contains information on how to support MPTCP natively in an
  app on Linux.
- `mptcp-info` contains info on how app devs can retrieve MPTCP info.
- `faq`, for Frequently Asked Questions.
- `apps`, the list of currently apps supporting MPTCP natively.
- `details` contains detailed explanations of the kernel implementation.

## Front matter
At the top of all pages a section between two `---` lines contains information
on the page.

All `jekyll` and `just the docs` features are supported, and new ones have been
added:
- `nav_titles` is a boolean. When set to true, the markdown titles of the page
  will be displayed in the navbar for easy access to a specific section.
- `titles_max_depth` is an integer that indicates the maximum level of titles
  included in the navbar. (*it directly corresponds to the number of* `#`)

## Liquid markers
- `{: .ctsm}`, can be used on `<details>` tags to display "`(click to see more)`"
  in grey text after the `summary`

- `{: warning}`, `{: note}`, `{: info}` are callouts. Used before or after a
  paragraph, they will highlight it.

## Mermaid
`mermaid` graphs are supported in markdown files.

## Contributions
This website is open to contributions via pull request. A link at the bottom
left of each page allows anyone to edit it on GitHub.
