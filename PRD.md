# Product Requirements Document (PRD)
## Articulation AI - Speech Lab

---

### 1. Product Overview
**Articulation AI** is a serverless, interactive web application designed to act as an AI-powered Pronunciation and Accent Coach. It bridges the gap between acoustic analysis (hearing) and biomechanics (physical movement) by showing users exactly how to move their mouth, jaw, and tongue to correct pronunciation errors.

The app uses Google's Gemini API as a "Clinical Ear" to analyze recorded audio against a target sentence, diagnose phonetic errors, and map them to physical movement instructions. These instructions are visualized on a high-fidelity sagittal head cross-section map.

---

### 2. User Experience & Critical User Journeys (CUJs)

#### CUJ 1: Sandbox exploration (Demo Mode)
*   **Goal**: Allow users to experience the core value of the app instantly without requiring setup (no API key needed).
*   **Flow**:
    1.  User opens the app and sees the "Interactive Speech Clinic" blank state.
    2.  User clicks **"Run Demo Diagnostic"**.
    3.  The UI instantly populates with a mock analysis for the sentence *"She sells seashells by the seashore"* where the user pronounced "sells" as "shells" (a common Alveolar-to-Palatal error).
    4.  The **Articulatory Map** displays the static medical head illustration.
    5.  The user clicks on the red-flagged word **"sells"** in the pitfalls list.
    6.  The map animates: it shows a red dot on the `Hard Palate` (where they incorrectly placed their tongue), draws a dashed cyan transition path, and slides a glowing particle to the `Alveolar Ridge` (the correct target), which then pulses green.
    7.  The user clicks **"View STAR Animation"** to open the SpeechSTAR reference video for this specific error comparison.

#### CUJ 2: Real-time Voice Practice (Active User Mode)
*   **Goal**: Allow users to practice custom sentences and receive personalized biomechanical feedback.
*   **Flow**:
    1.  User configures their API Key (either via URL parameter or Settings modal).
    2.  User selects a preset target sentence (e.g., sibilants focus, dental /th/ practice) or types a custom sentence.
    3.  User clicks the **microphone button**, speaks the sentence, and clicks stop.
    4.  The browser captures the audio, converts it to Base64, and sends it to the Gemini API.
    5.  The UI shows an "Analyzing..." loading state.
    6.  On response, the UI displays the overall accuracy score, transcription, and a list of phonetic pitfalls.
    7.  User interacts with the pitfalls to see the physical correction animated on the Articulatory Map.

#### CUJ 3: Easy Onboarding (URL Key Param)
*   **Goal**: Simplify the onboarding process for users (e.g., students or patients) by allowing teachers or clinics to distribute pre-configured links.
*   **Flow**:
    1.  User clicks a link containing the API key in the URL: `http://localhost:8080/index.html?key=YOUR_API_KEY`.
    2.  The app automatically extracts the key, saves it to `localStorage`, and updates the status to **"Gemini Ready"**.
    3.  The user can immediately start recording their voice.

---

### 3. Functional Requirements

#### 3.1. Audio Capture & Input
*   Capture audio from the user's microphone using the HTML5 Web Audio API and `MediaRecorder`.
*   Support recording start, stop, and play back.
*   Provide a real-time wave visualizer (CSS/JS animation) to show microphone activity while recording.

#### 3.2. Gemini AI Integration
*   Use `gemini-3.5-flash` as the default model for fast, low-latency audio processing.
*   Allow advanced users to switch to `gemini-3.1-pro-preview` or `gemini-2.5-flash-lite` in Settings.
*   Enforce a strict JSON schema for the Gemini response to ensure reliability:
    *   `transcription`: What was heard.
    *   `intended_sentence`: What was supposed to be said.
    *   `score`: Overall accuracy (0-100).
    *   `mispronounced_words`: Array of objects containing the `word`, `how_heard`, `problem`, `placement_guide`, `incorrect_articulator` (neutral/lips/tongue-tip/tongue-back/dental/palatal), `correct_articulator`, and `speechstar_url`.

#### 3.3. Articulatory Map (Visual Biomechanics)
*   Display a high-fidelity sagittal head cross-section image as the background of the canvas.
*   Crop the background image horizontally to isolate the head and hide auxiliary dashboard elements in the static image.
*   Draw animated overlays on top of the image to illustrate correction:
    *   **Dashed Trajectory Path**: A glowing dashed cyan line connecting the start point (incorrect) to the end point (correct).
    *   **Motion Particle**: A glowing particle with a bright white core that slides along the trajectory path.
    *   **Color Transition**: The particle interpolates from red (start/error) to green (end/correct) during motion.
    *   **Pulsing Target Dot**: The final target pulses green during the hold phase. The incorrect starting point pulses red during the transition phase.
    *   **Static Guides**: When no animation is running, show target guides as subtle grey dashed circles.
*   Provide manual trigger buttons to preview individual articulation points (`Tongue Tip`, `Lips`, `Jaw/Dental`, `Velum`).

#### 3.4. SpeechSTAR Portal Alignment
*   Extract reference IDs for common phonetic shifts (e.g., *s vs sh* = 34, *s vs th* = 36).
*   Generate direct deep-links to the SpeechSTAR portal using these IDs.
*   Provide a button in the UI that opens the SpeechSTAR link, which automatically opens the correct comparison video on load.

#### 3.5. Settings & Accents
*   Provide a Settings modal to configure:
    *   API Key.
    *   Gemini Model.
    *   Reference Accent (US or UK) - passed as context to Gemini.
*   Persist all settings in browser `localStorage`.
*   Support automatic model migration (e.g., transition old legacy model names to `gemini-3.5-flash`).

#### 3.6. Practice History
*   Store past practice runs (date, sentence, score, details) in `localStorage`.
*   Display the history list in a collapsible side panel or lower section.
*   Provide a option to clear history.

---

### 4. Technical Requirements & Architecture

*   **Architecture**: Single-file web application (`index.html`) + assets folder. Serverless and self-contained.
*   **Frontend Stack**:
    *   Tailwind CSS for utility-first responsive layout and glassmorphic styling.
    *   Vanilla ES6+ JavaScript for application logic, state management, and canvas animation loop.
    *   HTML5 Canvas 2D Context for real-time overlay drawing.
*   **Hosting**: Can be hosted on any static file host (GitHub Pages, Firebase Hosting, local HTTP server).
*   **Security**:
    *   API keys are stored strictly client-side in the user's browser `localStorage`.
    *   No backend logs or forwards the API key.
    *   Requests are sent directly from the client browser to the `generativelanguage.googleapis.com` domain.

---

### 5. Future Scope
*   **Phoneme-level scoring**: Provide detailed accuracy scores for individual sounds rather than just words.
*   **Intonation & Pitch Tracking**: Use Web Audio analysis to draw the user's pitch curve compared to a native speaker's model.
*   **Offline Mode**: Integrate local WebAssembly models (like whisper.cpp or small tensorflow models) to run without an internet connection or API key.
