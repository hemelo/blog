# Henrique Melo Blog

A personal developer blog built with [Astro](https://astro.build) to share knowledge, experiences, and insights from the world of software development.

## 🚀 About

This is the personal blog of **Henrique Melo**, a software engineer and developer passionate about sharing knowledge with the developer community. The blog serves as a platform to:

- Share technical insights and tutorials
- Document learning experiences
- Contribute to the developer community
- Showcase projects and achievements
- Provide valuable resources for fellow developers

## ✨ Features

- **Modern Tech Stack**: Built with Astro for optimal performance
- **Beautiful UI**: Clean, responsive design with Tailwind CSS
- **Rich Content Support**: 
  - Markdown and MDX support
  - LaTeX math rendering
  - Code syntax highlighting with 59+ color schemes
  - Reading time estimation
  - SEO optimization
- **Developer Experience**:
  - TypeScript support
  - Hot reload during development
  - Prettier code formatting
  - Comprehensive search functionality
- **Performance**: 
  - Static site generation
  - Image optimization
  - Sitemap generation
  - RSS feed support

## 🛠️ Tech Stack

- **Framework**: [Astro](https://astro.build) - Modern static site generator
- **Styling**: [Tailwind CSS](https://tailwindcss.com) - Utility-first CSS framework
- **Language**: [TypeScript](https://www.typescriptlang.org/) - Type-safe JavaScript
- **Content**: Markdown + MDX with custom plugins
- **Search**: [Pagefind](https://pagefind.app/) - Static search engine
- **Code Highlighting**: [Expressive Code](https://expressive-code.com/) with multiple themes
- **Math Rendering**: KaTeX for LaTeX support
- **Icons**: Custom emoji and icon support

## 🚀 Getting Started

### Prerequisites

- Node.js (version 18 or higher)
- npm or yarn package manager

### Installation

1. Clone the repository:
```bash
git clone <repository-url>
cd my-new-blog
```

2. Install dependencies:
```bash
npm install
```

3. Start the development server:
```bash
npm run dev
```

4. Open your browser and navigate to `http://localhost:4321`

### Available Scripts

- `npm run dev` - Start development server
- `npm run build` - Build for production
- `npm run preview` - Preview production build
- `npm run format` - Format code with Prettier

## 📁 Project Structure

```
my-new-blog/
├── src/
│   ├── components/     # Reusable UI components
│   ├── content/        # Blog posts and content
│   ├── layouts/        # Page layouts
│   ├── pages/          # Route pages
│   ├── plugins/        # Custom remark/rehype plugins
│   └── site.config.ts  # Site configuration
├── public/             # Static assets
├── astro.config.mjs    # Astro configuration
├── tailwind.config.js  # Tailwind CSS configuration
└── package.json        # Dependencies and scripts
```

## 🎨 Customization

### Themes
The blog includes 59+ beautiful color schemes including:
- GitHub themes (light/dark variants)
- Dracula, Nord, One Dark Pro
- Catppuccin variants
- Gruvbox themes
- And many more...

### Configuration
Edit `src/site.config.ts` to customize:
- Site metadata
- Navigation links
- Theme preferences
- Social media links
- Content settings

## 📝 Writing Posts

Create new blog posts in the `src/content/` directory using Markdown or MDX format. The blog supports:

- Frontmatter for metadata
- Custom admonitions and callouts
- GitHub-style code blocks
- Mathematical expressions
- Interactive components

## 🌐 Deployment

The blog is configured for static hosting and can be deployed to:
- Vercel
- Netlify
- GitHub Pages
- Any static hosting service

Build the project with:
```bash
npm run build
```

## 🤝 Contributing

While this is a personal blog, suggestions and feedback are welcome! Feel free to:
- Report bugs or issues
- Suggest improvements
- Share ideas for new features

## 📄 License

This project is licensed under the terms specified in the [LICENSE.txt](LICENSE.txt) file.

## 🔗 Links

- **Live Blog**: [blog.hemelo.fyi](https://blog.hemelo.fyi)
- **GitHub**: [@hemelo](https://github.com/hemelo)
- **Resume**: [resume.hemelo.fyi](https://resume.hemelo.fyi)

## 🙏 Acknowledgments

- **Theme**: Based on [MultiTerm Astro](https://github.com/stelcodes/multiterm-astro) by [@stelcodes](https://github.com/stelcodes) - A coder-ready Astro blog theme with 60+ color schemes
- Built with [Astro](https://astro.build) - The modern web framework for content-driven websites
- Styled with [Tailwind CSS](https://tailwindcss.com) - A utility-first CSS framework
- Icons and themes from the open-source community
