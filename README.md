# Technology Toolbox website

This repository contains the Technology Toolbox website:
[https://www.technologytoolbox.com](https://www.technologytoolbox.com).

![Technology Toolbox responsive website](https://assets.technologytoolbox.com/website/img/theme-screenshots/responsive-website-2300x1400.jpg)

## Features

- Built with [Hugo](https://gohugo.io)
- Responsive design (mobile friendly)
- Site structure designed for hundreds (or even thousands) of blog posts
  -- organized by year/month/day
- Multiple taxonomies for blog content (view posts by tags, categories, or date)
- [Bootstrap 5](https://getbootstrap.com/) (for styles and components)
- [Bootstrap Icons](https://icons.getbootstrap.com/) (for small, inline SVG
  images)
- [Commento](https://commento.io/) (for blog comments)
- Uses [Cloudflare](https://www.cloudflare.com/) content delivery network (CDN)
  for third-party libraries
- Improved Google Analytics to support
  [Content Security Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)
  for reducing the risk of cross-site scripting (XSS) attacks
- Syntax highlighting for code samples

## Installation

> Important: This website requires
> [installing the "extended" version of Hugo](https://gohugo.io/getting-started/installing/)
> in order to compile Sass/SCSS.

1. Clone the repository:

   ```Shell
   git clone https://github.com/technology-toolbox/website.git
   ```

1. Copy the theme to the website:

   ```Shell
   cd website

   git submodule add https://github.com/technology-toolbox/techtoolbox-hugo themes/techtoolbox-hugo
   ```

1. Install NPM dependencies:

   ```PowerShell
   # Install NPM dependencies for website theme
   pushd "themes/techtoolbox-hugo"

   npm install

   popd

   # Install NPM dependencies for website
   npm install
   ```

1. Start the Hugo server:

   ```Shell
   hugo server
   ```

   or

   ```Shell
   npm start
   ```

1. Browse the website: [http://localhost:1313/](http://localhost:1313/)

## Contributing

If you find a bug, have a question, or would like to submit a feature idea,
please use the
[issues list](https://github.com/technology-toolbox/website/issues).

If you would like to contribute, please
[fork the repository](https://github.com/technology-toolbox/website/fork) and
make the changes in a feature branch.
[Pull requests](https://github.com/technology-toolbox/website/pulls) are
welcome!

## License

This website is released under the
[MIT License](https://github.com/technology-toolbox/website/blob/main/LICENSE).
