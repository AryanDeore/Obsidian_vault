
# Dino Gesture Runner - Product Requirements Document (PRD)

## Overview

Dino Gesture Runner is a web-based game that lets users control Chrome's classic offline Dinosaur game using hand gestures captured via laptop webcam. Built with Gradio, MediaPipe Hands, and OpenCV, it provides real-time gesture recognition for jump (raise hand) and duck (lower hand) actions. The app deploys instantly for sharing, turning a local Python script into a fun, accessible ML demo that anyone can play in their browser—no installs required.

## Goals
- Create a shareable, zero-setup web demo of hand gesture control using pretrained MediaPipe models.
- Hook into Chrome Dino mechanics (jump/duck) via visual simulation or embedded game canvas.
- Enable quick deployment for portfolio showcase, tutorials, or social sharing.
- Target: Beginners in CV/ML wanting fun projects; fitness enthusiasts for gesture games; remote teams for icebreakers.
    
## Target Users
- ML hobbyists and students building webcam projects.
- Tech professionals (like CTOs) experimenting with real-time inference.
- Casual gamers preferring hands-free play.
- Constraints: Laptop with webcam; browser access; works on CPU (no GPU).

## Key Features
- **Live Webcam Feed**: Real-time hand tracking with gesture overlays (e.g., "JUMP" text).
- **Gesture Controls**:
- 
    |Gesture|Action|Trigger|
    |---|---|---|
    |Hand raised (wrist Y < 30% frame height)|Jump (spacebar emulation)|Dino hops over cactus|
    |Hand lowered (wrist Y > 70% frame height)|Duck (down arrow)|Dino crouches under pterodactyl|
    |No hand|Idle run|Default state|
    
- **Game Integration**: Embed HTML5 Dino clone canvas; gestures trigger JS events for jumps/ducks.
- **UI Elements**: Score display, action label, confidence meter, reset button.
- **Performance**: 30 FPS on standard laptops; low latency (<100ms per frame).

## Technical Requirements
## Core Stack
- **Frontend**: Gradio (webcam input, live outputs).
- **CV/ML**: MediaPipe Hands (pretrained landmark detection).
- **Processing**: OpenCV (frame handling), NumPy (coordinates).
- **Game**: Custom HTML5/JS Dino emulator (or iframe if feasible).

## MVP Functionality
1. Webcam capture → MediaPipe process → Landmark extraction.
2. Threshold-based classification (no custom training).
3. Visual feedback: Annotated video + game state.
    
4. Public share link via `iface.launch(share=True)`.
    

## Sample Code Structure

python

`# Main components (as discussed) - Hand detection: mp_hands.Hands(min_detection_confidence=0.7) - Jump logic: if wrist.landmark.y < 0.3 → trigger_jump() - Gradio Interface: inputs=gr.Image(webcam), outputs=[video, text, game_canvas]`

## User Flow

1. User visits share link → Gradio app loads.
    
2. Click "Play" → Webcam activates; game starts.
    
3. Show hand high → Dino jumps; low → Dino ducks.
    
4. Game progresses with score; restart on crash.
    
5. Share high score screenshot.
    

## Success Metrics

- **Engagement**: 80% users achieve score >500 in first try.
    
- **Performance**: <5% frame drops at 30 FPS.
    
- **Usability**: Works on 90% modern laptops/browsers.
    
- **Sharing**: 50+ plays via free Gradio link in 72 hours.
    

## Non-Functional Requirements

- **Deployment**: Free Hugging Face Spaces or Gradio share (no server costs).
    
- **Privacy**: Local processing only; no video storage.
    
- **Accessibility**: Thresholds adjustable for hand sizes/lighting.
    
- **Mobile**: Webcam support via desktop browsers (Chrome/Firefox).
    
- **Extensibility**: Easy swap for other gestures/games.
    

## Risks & Mitigations

|Risk|Mitigation|
|---|---|
|Lighting variance breaks detection|Add auto-exposure; fallback to color thresholding.|
|Browser game isolation|Use pure JS Dino clone instead of iframe.|
|Gradio webcam limits|Fallback to Streamlit if needed.|
|Hand occlusion|Limit to single hand; add tutorials.|

## Timeline (Solo Dev)

- Day 1: Gesture detection + basic Gradio UI.
    
- Day 2: Embed Dino game + tuning thresholds.
    
- Day 3: Polish UI, deploy, test sharing.
    

## Future Enhancements

- Custom gesture training via MediaPipe Model Maker.
    
- Multiplayer scoreboards.
    
- More games (Pong, Fruit Ninja).
    
- Voice feedback for actions.
    

This PRD captures a complete, deployable MVP from our conversation—fun, educational, and portfolio-ready. Start coding the Gradio script next!