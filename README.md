
# 📚 Dev Cheat Sheets Collection

> Your go-to library of quick references for professional development workflows

A curated collection of beautifully designed, print-friendly cheat sheets covering essential development workflows and best practices. Perfect for indie developers who want professional workflows without unnecessary complexity.

## 🚀 Live Demo

**[View Live Collection →](https://barbar0jav1er.github.io/dev-cheat-sheets)**

## 📋 What's Included

### 🔄 [GitHub Flow Cheat Sheet](github-flow-cheatsheet.html)
Complete visual guide to GitHub Flow - the perfect balance between simplicity and professionalism.

**Features:**
- ✅ Step-by-step workflow
- ✅ Essential commands
- ✅ Best practices & anti-patterns
- ✅ Special cases (hotfixes, PRs)
- ✅ CI/CD integration tips

**Perfect for:** Individual developers, small teams, simple projects

---

### 📦 [Semantic Versioning Cheat Sheet](semver-cheatsheet.html)
Master semantic versioning with practical examples and real-world scenarios.

**Features:**
- ✅ MAJOR.MINOR.PATCH rules
- ✅ Pre-releases (alpha, beta, RC)
- ✅ NPM & Git commands
- ✅ Practical examples
- ✅ Common anti-patterns to avoid

**Perfect for:** Any project that needs proper versioning

---

### 🔧 [GitHub Flow + Linter Integration](github-flow-linter.html)
Complete integration of linters into your workflow with automatic quality gates.

**Features:**
- ✅ 3-level linting strategy
- ✅ Pre-commit hooks setup
- ✅ CI/CD integration
- ✅ Branch protection rules
- ✅ Smart auto-merge

**Perfect for:** Quality-focused developers, production-ready code

---

### 🔀 [Hybrid GitFlow for Serious Indies](hybrid-gitflow-cheatsheet.html)
The best of both worlds: GitHub Flow simplicity + GitFlow control when you need it.

**Features:**
- ✅ Smart hybrid strategy
- ✅ Decision tree for workflow choice
- ✅ Pre-release timeline guide
- ✅ Real project examples
- ✅ KPIs and metrics

**Perfect for:** Serious indie developers, scalable workflows

## 🎯 Why These Cheat Sheets?

### 📊 Quick Stats
- **4** comprehensive cheat sheets
- **100+** commands and examples
- **∞** time saved
- **🚀** productivity boosted

### ✨ Key Benefits

- **🖨️ Print-Friendly:** Optimized for printing and PDF generation
- **📱 Responsive:** Looks great on any device
- **🎨 Visual:** Beautiful design with clear hierarchy
- **⚡ Practical:** Real commands, real examples
- **🔄 Updated:** Latest 2025 best practices
- **🚀 Ready-to-Use:** Copy-paste commands that work

## 🏃‍♂️ Quick Start

### Option 1: View Online
Just visit the [live demo](https://barbar0jav1er.github.io/dev-cheat-sheets) and start browsing!

### Option 2: Local Setup
```bash
# Clone the repository
git clone https://github.com/barbar0jav1er/dev-cheat-sheets.git

# Navigate to the directory
cd dev-cheat-sheets

# Open in your browser
open index.html
# or
python -m http.server 8000  # Then visit http://localhost:8000
```

### Option 3: Deploy Your Own
Deploy to any static hosting service:

**GitHub Pages:**
1. Fork this repository
2. Go to Settings → Pages
3. Select source: Deploy from branch `main`
4. Your site will be live at `https://barbar0jav1er.github.io/dev-cheat-sheets`

**Netlify:**
1. Drag the project folder to [netlify.com/drop](https://netlify.com/drop)
2. Get instant live URL

**Vercel:**
1. Import project from GitHub
2. Deploy with one click

## 📖 How to Use

### For Developers
- **Browse online** for quick reference
- **Print individual sheets** for offline use
- **Bookmark specific sections** you use most
- **Share with your team** for consistent workflows

### For Teams
- **Standardize workflows** across team members
- **Onboard new developers** with clear guides
- **Reference during code reviews**
- **Print for office walls** (seriously, they look great!)

## 🛠️ Built With

- **HTML5** - Semantic structure
- **CSS3** - Modern styling with gradients and animations
- **Vanilla JavaScript** - Lightweight interactions
- **Inter Font** - Clean, professional typography
- **Responsive Design** - Mobile-first approach
- **Print Optimization** - Perfect for physical copies

## 📋 File Structure

```
dev-cheat-sheets/
├── index.html                      # Main landing page
├── github-flow-cheatsheet.html     # GitHub Flow guide
├── semver-cheatsheet.html          # Semantic Versioning guide  
├── github-flow-linter.html         # Linter integration guide
├── hybrid-gitflow-cheatsheet.html  # Hybrid GitFlow guide
├── README.md                       # This file
└── assets/                         # (Optional) Images, icons
```

## 🎨 Customization

### Adding New Cheat Sheets

1. **Create new HTML file** following the existing template structure
2. **Add card to index.html:**
```html
<div class="cheatsheet-card" onclick="window.open('your-new-cheatsheet.html', '_blank')">
    <div class="card-icon">🆕</div>
    <h3 class="card-title">Your New Cheat Sheet</h3>
    <!-- Add description, features, tags, and actions -->
</div>
```
3. **Update stats** in the stats section if needed

### Customizing Design

The CSS uses custom properties for easy theming:
```css
:root {
    --primary-color: #3b82f6;
    --secondary-color: #1e40af;
    --accent-color: #f59e0b;
    /* Modify these to match your brand */
}
```

## 🤝 Contributing

Contributions are welcome! Here's how you can help:

### 🐛 Found a Bug?
- Open an issue with details
- Include browser/device info
- Screenshots are helpful

### 💡 Have an Idea?
- Check existing issues first
- Open a new issue with your suggestion
- Explain the use case

### 🔧 Want to Contribute Code?
1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

### 📝 Improving Documentation?
- Fix typos
- Add examples
- Improve explanations
- Update outdated information

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- **GitHub** for amazing documentation and tools
- **Semantic Versioning** community for clear standards
- **Developer community** for best practices and feedback
- **Open source projects** that inspired these workflows

## 📞 Support

- **Issues:** [GitHub Issues](https://github.com/barbar0jav1er/dev-cheat-sheets/issues)
- **Discussions:** [GitHub Discussions](https://github.com/barbar0jav1er/dev-cheat-sheets/discussions)
- **Documentation:** Check individual cheat sheets for detailed guides

## 🔗 Useful Links

- [GitHub Flow Documentation](https://docs.github.com/en/get-started/quickstart/github-flow)
- [Semantic Versioning](https://semver.org)
- [Conventional Commits](https://conventionalcommits.org)
- [GitHub CLI](https://cli.github.com)
- [Semantic Release](https://github.com/semantic-release/semantic-release)

---

<div align="center">

**⭐ Star this repository if you find it helpful!**

*Made with 💙 for the developer community*

[🌟 Star](https://github.com/barbar0jav1er/dev-cheat-sheets/stargazers) · [🐛 Report Bug](https://github.com/barbar0jav1er/dev-cheat-sheets/issues) · [💡 Request Feature](https://github.com/barbar0jav1er/dev-cheat-sheets/issues)

</div>