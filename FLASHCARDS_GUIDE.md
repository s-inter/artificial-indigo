# Go Interview Flashcards - User Guide

## 🎯 Overview

This is an interactive flashcard system designed to help you prepare for Go technical interviews in a DevOps context. It contains 25 carefully curated questions covering basic to advanced Go concepts.

## 🚀 Quick Start

### Option 1: Use GitHub Pages (Recommended)

1. Enable GitHub Pages for this repository:
   - Go to repository Settings
   - Navigate to "Pages" section
   - Select source: "Deploy from a branch"
   - Choose branch: `main` (or your current branch)
   - Click Save

2. Access the flashcards at:
   ```
   https://<your-username>.github.io/artificial-indigo/
   ```

### Option 2: Run Locally

1. Clone the repository:
   ```bash
   git clone https://github.com/s-inter/artificial-indigo.git
   cd artificial-indigo
   ```

2. Open `index.html` in your web browser:
   ```bash
   # On macOS
   open index.html
   
   # On Linux
   xdg-open index.html
   
   # On Windows
   start index.html
   ```

   Or use a local web server:
   ```bash
   # Python 3
   python -m http.server 8000
   
   # Python 2
   python -m SimpleHTTPServer 8000
   
   # Node.js (with npx)
   npx http-server
   ```
   
   Then navigate to `http://localhost:8000`

## 📚 Features

### Interactive Learning
- **Flip Cards**: Click on a card or press Space to reveal the answer
- **Navigation**: Use arrow keys (← →) or click buttons to move between cards
- **Mark Studied**: Press Enter or click the button to track your progress

### Filtering Options
- **All**: View all 25 questions
- **Basic**: Focus on fundamental concepts (3 questions)
- **Intermediate**: Practice intermediate topics (21 questions)
- **Advanced**: Challenge yourself with advanced questions (1 question)

### Study Tools
- **Shuffle**: Randomize the order of cards
- **Study Mode**: Focus only on cards you haven't studied yet
- **Progress Tracking**: Your progress is saved in the browser's local storage
- **Reset Progress**: Start fresh anytime

### Keyboard Shortcuts
- `←` - Previous card
- `→` - Next card
- `Space` - Flip card
- `Enter` - Mark card as studied

## 📖 Content

### Topics Covered

The flashcards cover essential Go concepts for DevOps engineers:

1. **Language Fundamentals**
   - `make` vs `new`
   - Zero values
   - `defer` statement
   - `iota` constants
   - Struct tags

2. **Concurrency**
   - Goroutines and GMP model
   - Channels (buffered vs unbuffered)
   - `select` statement
   - Race conditions
   - Worker pools

3. **Memory & Performance**
   - Slices and arrays
   - Value vs pointer receivers
   - Performance optimization
   - Preventing goroutine leaks

4. **Error Handling & Best Practices**
   - Error handling philosophy
   - Testing strategies
   - Context package
   - Graceful shutdown

5. **Interfaces & Types**
   - Interface satisfaction
   - Empty interface (`any`)
   - `sync.Mutex` vs `sync.RWMutex`

6. **DevOps-Specific**
   - HTTP server configuration
   - Docker image building
   - Production logging
   - Go modules

## 📝 Study Guide

For comprehensive answers and code examples, refer to:
- **`interview_go_devops.md`**: Core Go concepts and topics overview
- **`go_interview_25qa.md`**: Full 25 Q&A with detailed explanations and code examples
- **`flashcards.json`**: Structured data used by the web interface

## 🛠️ Files in This Repository

- **`index.html`**: Interactive flashcard web application
- **`flashcards.json`**: Flashcard data in JSON format
- **`interview_go_devops.md`**: Comprehensive topic outline and concept overview
- **`go_interview_25qa.md`**: 25 interview questions with detailed answers and code examples
- **`FLASHCARDS_GUIDE.md`**: This file - user guide for the flashcard system
- **`README.md`**: Repository overview

## 💡 Tips for Effective Study

1. **Start with Basics**: Use the difficulty filters to progress gradually
2. **Understand, Don't Memorize**: Focus on understanding concepts rather than memorizing answers
3. **Practice Regularly**: Review cards daily for better retention
4. **Relate to Experience**: Connect concepts to your DevOps work
5. **Code Along**: Try implementing examples from the answers
6. **Use Study Mode**: Focus on cards you haven't mastered yet
7. **Track Progress**: Use the progress indicators to monitor your learning

## 🎓 Interview Preparation Strategy

1. **Week 1-2**: Go through all cards, mark familiar topics
2. **Week 3**: Focus on intermediate and advanced cards
3. **Week 4**: Use study mode to review difficult concepts
4. **Before Interview**: Quick review of all cards in shuffle mode

## 🔧 Customization

### Adding Your Own Cards

Edit `flashcards.json` to add more questions:

```json
{
  "id": 26,
  "question": "Your question here?",
  "answer": "Your detailed answer here.",
  "difficulty": "intermediate"
}
```

### Modifying Styles

The `index.html` file contains embedded CSS. You can customize:
- Colors (search for color values in the `<style>` section)
- Fonts (modify `font-family` properties)
- Layout (adjust padding, margins, sizes)

## 🐛 Troubleshooting

**Cards not loading?**
- Ensure `flashcards.json` is in the same directory as `index.html`
- Check browser console for errors (F12)
- Verify JSON syntax is valid

**Progress not saving?**
- Check that cookies/local storage is enabled in your browser
- Try a different browser
- Clear browser cache and reload

**Keyboard shortcuts not working?**
- Click on the page to ensure it has focus
- Check that no browser extension is intercepting the keys

## 📱 Mobile Support

The flashcard interface is fully responsive and works on:
- Desktop browsers
- Tablets
- Smartphones

Touch gestures are supported for flipping cards and navigation.

## 🤝 Contributing

Feel free to:
- Add more questions
- Improve existing answers
- Enhance the UI/UX
- Report issues
- Suggest new features

## 📄 License

This study material is provided as-is for educational purposes.

## 🙏 Acknowledgments

Based on Go best practices from:
- [Official Go Documentation](https://go.dev/doc/)
- [Effective Go](https://go.dev/doc/effective_go)
- [Go Blog](https://go.dev/blog/)
- DevOps community best practices

---

**Good luck with your interview preparation! 🚀**
