# UI Engineer Coding Test

Welcome! This is a take-home coding test designed to evaluate your visual design skills, attention to detail, and Vue.js proficiency. **Expected time: 2-4 hours.**

## Your Mission

You'll be working with a responsive page template system. We've built a basic starter implementation, but it needs your design expertise to make it shine. Your job is to **redesign and polish the `GenericTemplate` component** to work beautifully on both mobile AND desktop.

## What's Already Built

<img width="4098" height="2290" alt="Screenshot 2025-11-04 at 12 33 22 pm" src="https://github.com/user-attachments/assets/177f6852-50ea-4d2e-868f-a83aae3b4ecc" />


This project includes:
- A Vue 2.7 app with Vite
- Side-by-side mobile (375px) and desktop (1200px+) preview panels
- Three page types: Program, Event, and Sales Page (switch using tabs)
- A basic `GenericTemplate` component that accepts:
  - `title` - Page title
  - `blocks` - Array of content blocks (hero images, text, videos)
  - `actions` - Array of CTA buttons with:
    - `button_text` - The default button text
    - `pricing_text` - Optional pricing/subtitle text
    - `post_join_text` - The button text after clicking (e.g., "Joined", "Registered")
- Interactive state management:
  - Bookmark button toggles on/off
  - CTA button changes text after clicking
  - State persists to localStorage per page

The current implementation is intentionally basic—**it's your canvas to improve**.

## Your Focus Areas

### 1. **Make the Pinned CTA More Prominent and Engaging** ⭐
The call-to-action section is currently bland. Make it pop! Consider:
- Visual hierarchy and emphasis
- How pricing text relates to the button
- Micro-interactions and states (the button already changes on click!)
- Different visual treatment for the "purchased" state
- Making users *want* to click

### 2. **Design a Toolbar That Works on Desktop** 🖥️
The current toolbar is mobile-first (back button, title, bookmark). On desktop:
- How should the layout change?
- Should it be persistent, or can it be more elegant?
- How could it feel more native for desktop users, and less like a mobile app expanded to desktop?
- The bookmark button already has interactive state - how can you enhance it?

### 3. **Add Polish** ✨
Details matter. Consider:
- Transitions and animations
- Hover states
- Loading states (images, videos)
- Scroll behavior and progress
- Typography and spacing refinement
- Visual feedback for all interactive elements

### 4. **Ensure Flexibility Across All Three Page Types** 🔄
Your design must work elegantly for:
- **Program** - Product page with pricing
- **Event** - Event registration with video
- **Sales Page** - Membership signup with trial offer

Think in systems, not one-offs.

## What We're Evaluating

1. **Visual Design & Polish** - Does it look and feel premium?
2. **Attention to Detail** - Did you sweat the small stuff?
3. **Responsive Design Thinking** - Does it adapt intelligently to mobile vs. desktop?
4. **Code Quality** - Is your code clean, well-organized, and maintainable?
5. **Systems Thinking** - Does your solution work across all use cases?

## Technical Details

- **Vue Version**: 2.7 (supports both Options API and Composition API)
- **Styling**: Tailwind CSS is already set up if you want, but you can use plain CSS if you prefer
- **Component Structure**: You can modify the component structure, but it must still accept the same props (title, blocks, actions)

## Deliverables

1. **Improved `GenericTemplate.vue` component**
   - Enhanced visual design
   - Responsive behavior for mobile and desktop
   - Polish and micro-interactions

2. **Updated styles**

3. **A 2-3 minute Loom video** explaining:
   - Your design decisions
   - Trade-offs you considered
   - What you'd do with more time
   - Record at: https://loom.com

## Tips for Success

- **Polish over features** - We'd rather see one thing done beautifully than five things done halfway
- **Think about the feel, not just the look** - How does it feel to use?
- **Show your taste** - This is your chance to show us what good design means to you

## Getting Started

1. **Install dependencies:**
   ```bash
   npm install
   ```

2. **Run the dev server:**
   ```bash
   npm run dev
   ```

3. **Open your browser:**
   Navigate to `http://localhost:3000`

4. **Start designing!**
   Edit `src/components/GenericTemplate.vue`

5. **Try the interactive features:**
   - Click the bookmark icon to toggle the bookmarked state
   - Click the CTA button to see it change to the "purchased" state
   - Switch between tabs and see that state is preserved per page

## Project Structure

```
coding-test/
├── src/
│   ├── components/
│   │   └── GenericTemplate.vue  ← Your main work happens here
│   ├── App.vue                  ← Preview container (you can tweak if needed)
│   ├── main.js
│   └── style.css                ← Global styles and CSS variables
├── index.html
├── package.json
└── README.md                    ← You are here
```

## Page Data Reference

### Program Page
- **Title**: "Amari Fitness Program"
- **CTA**: "Join Program" → "Joined" (after click)
- **Pricing**: "From $197/month"
- **Content**: Hero image, program description, benefits list

### Event Page
- **Title**: "Live Yoga Session"
- **CTA**: "Register Now" → "Registered" (after click)
- **Pricing**: "Free"
- **Content**: Hero image, event details, video embed, what to bring

### Sales Page
- **Title**: "Premium Membership"
- **CTA**: "Start Free Trial" → "Trial Started" (after click)
- **Pricing**: "Then $29/month"
- **Content**: Hero image, membership pitch, feature list, trial terms

## Submission

When you're done:
1. Record your 2-3 minute Loom video
2. Push it to a public repo, or share a `.zip` file with the project
3. Message me on Upwork with the loom video & code

## Questions?

If you have questions about requirements or run into technical issues, please reach out to me on Upwork.

---

**Good luck! I'm excited to see what you create.** 🚀
