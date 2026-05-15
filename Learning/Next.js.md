1. add header and footer in `layout.tsx` 
2. src/app folder for routing and src/components for components


```
# Interactive Portfolio Website - Complete Project Documentation

  

## Project Overview

  

This is an interactive portfolio website with an AI-powered chat interface that allows visitors to ask questions about the portfolio owner's experience, skills, projects, and background. The website features a smooth transition between a landing page view and a conversational chat interface. Write tests.

  

### Key Features

- Interactive chat interface for portfolio exploration

- Smooth mode transitions (landing page ↔ chat mode)

- Sticky chat input that appears on scroll

- Responsive design for all screen sizes

- Professional projects section

- Footer with contact information

  

---

  

## Technology Stack

  

### Core Framework

- **Next.js latest**

- **React 18+** with TypeScript

- **Tailwind CSS v4** for styling

- **ESLint** for linting

- **shadcn/ui** as the UI library

  
  

### Color Palette

The design uses a clean, professional color scheme:

  
  

### Visual Layout Flow

  

**Landing Page Mode:**

\`\`\`

┌─────────────────────────────────────┐

│ Navbar (Fixed) │

├─────────────────────────────────────┤

│ │

│ Hero Section │

│ (Title, Chat Input) │

│ │

├─────────────────────────────────────┤

│ │

│ Testimonials Section │

│ (4 Cards Grid) │

│ │

├─────────────────────────────────────┤

│ │

│ Footer │

│ (Links, Contact, Social) │

│ │

└─────────────────────────────────────┘

\`\`\`

  

**Chat Mode:**

\`\`\`

┌─────────────────────────────────────┐

│ Navbar (Fixed, with Reset) │

├─────────────────────────────────────┤

│ │

│ │

│ Chat Messages │

│ (User & AI Messages) │

│ │

│ │

├─────────────────────────────────────┤

│ Chat Input (Fixed Bottom) │

└─────────────────────────────────────┘

\`\`\`

  

---

  

**Layout Structure:**

\`\`\`

┌─────────────────────────────────────────┐

│ │

│ Alex Johnson │

│ Full Stack Developer │

│ │

│ Ask me anything about my experience...│

│ │

│ [Chat Input Component] │

│ │

└─────────────────────────────────────────┘

\`\`\`

  

## User Interactions & Flows

  

### Flow 1: Landing Page → Chat Mode

  

\`\`\`

1. User lands on page

└── Sees: Hero, Projects, Footer

  

2. User types question in hero input

└── Can click example question or type custom

  

3. User clicks Send button

├── Question appears as user message in a pop-over modal

├── AI response appears below in the pop-over

└── View switches to chat mode

  

4. Chat mode active

├── Only navbar and chat visible

├── Can ask follow-up questions

└── "New Chat" button in navbar

\`\`\`

  

### Flow 2: Sticky Chat Input

  

\`\`\`

1. User on landing page

└── Scrolls down past hero section

  

2. Scroll position > 80vh

└── Sticky chat input appears at bottom

  

3. User types in sticky input

└── Same behavior as hero input

└── Switches to chat mode

\`\`\`

  

### Flow 3: Reset Chat

  

\`\`\`

1. User in chat mode

└── Clicks "New Chat" button in navbar

  

2. Application resets

├── Messages cleared to simulate new chat

└── Scroll position reset

\`\`\`

  

---

  
  

## Testing Checklist

  

### Functional Testing

- [ ] Chat input accepts text

- [ ] Example questions populate input

- [ ] Submit button disabled when empty

- [ ] Messages appear in correct order

- [ ] AI responses match keywords

- [ ] Reset button clears chat

- [ ] Sticky input appears on scroll

- [ ] Auto-scroll works in chat mode

  

### Visual Testing

- [ ] Gradients render correctly

- [ ] Shadows appear on hover

- [ ] Borders are consistent

- [ ] Spacing is uniform

- [ ] Icons are properly sized

- [ ] Avatars display correctly

- [ ] Cards align properly

  

### Responsive Testing

- [ ] Mobile layout (< 768px)

- [ ] Tablet layout (768px - 1024px)

- [ ] Desktop layout (> 1024px)

- [ ] Text remains readable at all sizes

- [ ] Buttons are tappable on mobile

- [ ] Grid columns adjust properly

  

### Browser Testing

- [ ] Chrome

- [ ] Firefox

- [ ] Safari

- [ ] Edge

- [ ] Mobile browsers

  

---
```