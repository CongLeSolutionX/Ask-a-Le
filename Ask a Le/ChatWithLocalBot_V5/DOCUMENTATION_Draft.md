---
created: 2025-05-12 05:31:26
author: Cong Le
version: "1.0"
license(s): MIT, CC BY 4.0
copyright: Copyright (c) 2025 Cong Le. All Rights Reserved.
---


# Semantic Review and Visual Explanation of `ChatWithLocalBot_V5.swift`
# 
> **Disclaimer:**
>
> This document contains my personal notes on the topic,
> compiled from publicly available documentation and various cited sources.
> The materials are intended for educational purposes, personal study, and reference.
> The content is dual-licensed:
> 1. **MIT License:** Applies to all code implementations (Swift, Mermaid, and other programming languages).
> 2. **Creative Commons Attribution 4.0 International License (CC BY 4.0):** Applies to all non-code content, including text, explanations, diagrams, and illustrations.
---


Below is a thorough semantic analysis of the given Swift file, followed by a **collection of concise, interconnected Mermaid diagrams**. These diagrams break down complex flows, architectures, and interactive patterns in the code, covering UI/UX, AI/voice integration, permission and state management, and 
error handling.

---

## 1. Semantic Review: Key Components and Flow

### A. Architectural Overview

- **`AICardView`** is a SwiftUI View that serves as a chat interface for asking questions *by voice* in Vietnamese, and then relaying the response from a local LLM (Large Language Model).
- **Voice functionality** is two-fold:
    - **Speech-to-Text**: Converts user's spoken Vietnamese question to text via **Speech framework** (SFSpeechRecognizer & AVAudioEngine).
    - **Text-to-Speech**: Reads out the AI's answer aloud using **AVSpeechSynthesizer**.
- The **AI logic** uses a local model (TinyLlama/Arcee-VyLinh, quantized) loaded via a `LLM` abstraction and queried asynchronously.
- Extensive **state management** ensures:
    - User's question and AI's answer are displayed and voiced,
    - Loading, error, and permission states are clearly presented to the user.
- **Permissions** for both speech recognition and microphone usage are requested and handled.
- **Error handling** is structured and user-facing, using custom error types (`AIError`) with localized descriptions.

### B. Core Functional Flow

1. **User taps mic button**:
    - If not recording:
        - Microphone and speech permissions checked/requested.
        - Recording starts and transcribes user's question in Vietnamese.
    - If recording:
        - Recording stops, final transcription is produced.
2. **On final transcription**:
    - The transcribed question is sent asynchronously to the local LLM.
    - Spinner/progress shows during processing.
    - AI's response is displayed AND spoken aloud via TTS.
3. **Error management**:
    - At any stage (permission denied, recording issues, LLM errors), clear error messages appear in the card.

---

## 2. Visual Explanations (Mermaid Diagrams)


### A. Top-Level App Architecture

```mermaid
---
title: "Top-Level App Architecture"
author: "Cong Le"
version: "1.0"
license(s): "MIT, CC BY 4.0"
copyright: "Copyright (c) 2025 Cong Le. All Rights Reserved."
config:
  layout: dagre
  theme: base
---
%%%%%%%% Mermaid version v11.4.1-b.14
%%%%%%%% Available curve styles include the following keywords:
%% basis, bumpX, bumpY, cardinal, catmullRom, linear, monotoneX, monotoneY, natural, step, stepAfter, stepBefore.
%%{
  init: {
    'flowchart': { 'htmlLabels': false, 'curve': 'linear' },
    'fontFamily': 'Fantasy,Monaco',
    'themeVariables': {
      'primaryColor': '#2FB1',
      'primaryTextColor': '#F8B229',
      'lineColor': '#F8B229',
      'primaryBorderColor': '#27AE60',
      'secondaryColor': '#EEF2',
      'secondaryTextColor': '#6C3483',
      'secondaryBorderColor': '#A569BD',
      'fontSize': '20px'
    }
  }
}%%
flowchart TD
    AICardView["AICardView<br/>(SwiftUI Card)"]
    subgraph VoiceInteraction
        SR["Speech-to-Text<br/>(SFSpeechRecognizer, AVAudioEngine)"]
        TTS["Text-to-Speech<br/>(AVSpeechSynthesizer)"]
    end
    AI["Local LLM<br/>(TinyLlama /<br/> Arcee-VyLinh via LLM API)"]
    %% User["User<br/>(Speaks Question)"]

	My_Meme@{ img: "https://raw.githubusercontent.com/CongLeSolutionX/MY_GRAPHIC_ASSETS/refs/heads/Designing_graphic_syntax/MY_MEME/My-meme-icon-design.png", label: "User<br/>(Speaks Question)", pos: "b", w: 100, h: 100, constraint: "on" }


    Permissions["Permission Handling<br/>(Speech & Microphone)"]
    UIState["UI State Management<br/>(Loading, Errors, Display)"]

    My_Meme -->|"Taps mic, speaks"| AICardView
    AICardView --> Permissions
    AICardView --> SR
    SR -->|"transcription"| AICardView
    AICardView -->|"query"| AI
    AI -->|"AI answer"| AICardView
    AICardView --> TTS
    TTS -->|"audio out"| My_Meme
    AICardView --> UIState
    UIState --> AICardView
    
```

- **Legend**: All flows are mediated by the `AICardView`, which orchestrates user input, permissions, AI calls, and output.

---

### B. UI State Machine: What the User Sees

```mermaid
---
title: "UI State Machine: What the User Sees"
config:
  look: handDrawn
  theme: base
---
%%%%%%%% Mermaid version v11.4.1-b.14
%%%%%%%% Available curve styles include the following keywords:
%% basis, bumpX, bumpY, cardinal, catmullRom, linear, monotoneX, monotoneY, natural, step, stepAfter, stepBefore.
%%{
  init: {
    'stateDiagram-v2': { 'htmlLabels': false},
    'fontFamily': 'Fantasy,Monaco',
    'themeVariables': {
      'primaryColor': '#B528',
      'primaryTextColor': '#F8B229',
      'primaryBorderColor': '#7C33',
      'lineColor': '#F8B229',
      'secondaryColor': '#3312',
      'tertiaryColor': '#fff',
      'fontSize': '20px'
    }
  }
}%%
stateDiagram-v2
  [*] --> Idle
  Idle : "Nhấn nút micro để hỏi" <br/> ("Press the microphone button to ask.")
  Idle --> Recording : Mic tapped
  Recording : "Đang nghe..." <br/> ("Listening...")
  Recording --> AIProcessing : Recording stopped or transcription final
  AIProcessing : "Đang xử lý..." <br/> ("Processing...")
  AIProcessing --> Answered : AI response received
  AIProcessing --> Error : AI error or empty result
  Recording --> Error : Speech permission denied OR mic error
  AnyState --> Error : Any error
  Error : Error banner/<br/>message shown
  Answered : Shows "AI trả lời" <br/>and reads aloud
  Error --> Idle : User retries/<br/>discards error
  Answered --> Idle : User asks new question
  
```

---

### C. Detailed Sequence: Voice-to-AI-to-Voice Pipeline

```mermaid
---
title: "Voice-to-AI-to-Voice Pipeline"
author: "Cong Le"
version: "0.1"
license(s): "MIT, CC BY 4.0"
copyright: "Copyright (c) 2025 Cong Le. All Rights Reserved."
config:
  layout: elk
  theme: base
---
%%%%%%%% Mermaid version v11.4.1-b.14
%%{
  init: {
    'sequence': { 'mirrorActors': true, 'showSequenceNumbers': true, 'actorMargin': 50 },
    'fontFamily': 'Fantasy,Monaco',
    'themeVariables': {
      'primaryColor': '#2BB8',
      'primaryBorderColor': '#7C0000',
      'lineColor': '#F8B229',
      'secondaryColor': '#6122',
      'tertiaryColor': '#fff',
      'fontSize': '15px',
      'textColor': '#F8B229',
      'actorTextColor': '#E2E',
      'fontSize': '20px',
      'stroke':'#033',
      'stroke-width': '0.2px'
    }
  }
}%%
sequenceDiagram
    actor User
    participant UI as AICardView
    participant Perm as Permissions
    participant Rec as SpeechRecog
    participant LLM as Local AI
    participant TTS as TextToSpeech

    User->>UI: Tap mic
    UI->>Perm: Check/request permissions
    Perm-->>UI: Result (authorized/denied)
    alt Permission granted
        UI->>Rec: Start recording
        Rec-->>UI: Live transcription (updates UI)
        User->>UI: Tap stop OR auto-stop
        Rec-->>UI: Final transcription
        UI->>LLM: Query AI with question
        LLM-->>UI: AI answer
        UI->>TTS: Speak answer
        TTS-->>User: Audio
    else Permission denied/error
        Perm-->>UI: Show error message
    end
```

---

### D. AIModel Loading and Error Handling

```mermaid
---
title: "AI Model Loading and Error Handling"
author: "Cong Le"
version: "1.0"
license(s): "MIT, CC BY 4.0"
copyright: "Copyright (c) 2025 Cong Le. All Rights Reserved."
config:
  layout: dagre
  theme: base
---
%%%%%%%% Mermaid version v11.4.1-b.14
%%%%%%%% Available curve styles include the following keywords:
%% basis, bumpX, bumpY, cardinal, catmullRom, linear, monotoneX, monotoneY, natural, step, stepAfter, stepBefore.
%%{
  init: {
    'flowchart': { 'htmlLabels': false, 'curve': 'linear' },
    'fontFamily': 'Fantasy,Monaco',
    'themeVariables': {
      'primaryColor': '#2FB1',
      'primaryTextColor': '#F8B229',
      'lineColor': '#F8B229',
      'primaryBorderColor': '#27AE60',
      'secondaryColor': '#EE22',
      'secondaryTextColor': '#6C3483',
      'secondaryBorderColor': '#A569BD',
      'fontSize': '20px'
    }
  }
}%%
flowchart LR
	subgraph AI_Interaction["AI Interaction"]
		A1("runDemoAIModel(question)")
        A2{Model loads successfully?}
        A3["-> Preprocess question"]
        A4["-> Get completion from AI"]
        A5["<- AI returns response"]
        A6["Return response to view"]
    end
    subgraph Errors["Errors"]
        E1["Initialization failure"]
        E2["Processing error"]
        E3["Speech recognition error"]
        E4["Permission denied"]
    end

    A1 --> A2
    A2 -- Yes --> A3
    A3 --> A4 --> A5 --> A6
    A2 -- No --> E1
    A4 -- Error --> E2
    A1 -- "Error<br/>(speech recog)" --> E3
    A1 -- "Error<br/>(permission)" --> E4
```

---

### E. Speech Recognition Flow

```mermaid
---
title: "Speech Recognition Flow"
author: "Cong Le"
version: "1.0"
license(s): "MIT, CC BY 4.0"
copyright: "Copyright (c) 2025 Cong Le. All Rights Reserved."
config:
  layout: dagre
  theme: base
---
%%%%%%%% Mermaid version v11.4.1-b.14
%%%%%%%% Available curve styles include the following keywords:
%% basis, bumpX, bumpY, cardinal, catmullRom, linear, monotoneX, monotoneY, natural, step, stepAfter, stepBefore.
%%{
  init: {
    'flowchart': { 'htmlLabels': false, 'curve': 'linear' },
    'fontFamily': 'Fantasy,Monaco',
    'themeVariables': {
      'primaryColor': '#2FB1',
      'primaryTextColor': '#F8B229',
      'lineColor': '#F8B229',
      'primaryBorderColor': '#27AE60',
      'secondaryColor': '#EDE2',
      'secondaryTextColor': '#6C3483',
      'secondaryBorderColor': '#A569BD',
      'fontSize': '20px'
    }
  }
}%%
flowchart TD
	My_Meme@{ img: "https://raw.githubusercontent.com/CongLeSolutionX/MY_GRAPHIC_ASSETS/refs/heads/Designing_graphic_syntax/MY_MEME/My-meme-icon-design.png", label: "User taps mic", pos: "b", w: 100, h: 100, constraint: "on" }
	
    My_Meme --> CheckPerm["Check Speech &<br/> Mic Permissions"]
    CheckPerm -- authorized --> SetupAudio["Setup AVAudioEngine"]
    SetupAudio --> StartRecogSession["Create recognition request & task"]
    StartRecogSession --> Listening["Listen & transcribe..."]
    Listening -- partial results --> UpdateUI["Update displayed question"]
    Listening -- user stops/auto final --> Finalize["Stop recording"]
    Finalize --> IsTextFinal["Is transcription final <br/>& non-empty?"]
    IsTextFinal -- Yes --> FetchAI["Call AI with question"]
    IsTextFinal -- No --> ShowError["Show 'no voice detected' error"]
    CheckPerm -- "denied/<br/>error" --> ShowError
    Listening -- mic/audio error --> ShowError
```

---

### F. Error Handling Map

```mermaid
---
title: "Error Handling Map"
author: "Cong Le"
version: "1.0"
license(s): "MIT, CC BY 4.0"
copyright: "Copyright (c) 2025 Cong Le. All Rights Reserved."
config:
  layout: dagre
  theme: base
---
%%%%%%%% Mermaid version v11.4.1-b.14
%%%%%%%% Available curve styles include the following keywords:
%% basis, bumpX, bumpY, cardinal, catmullRom, linear, monotoneX, monotoneY, natural, step, stepAfter, stepBefore.
%%{
  init: {
    'flowchart': { 'htmlLabels': false, 'curve': 'linear' },
    'fontFamily': 'Fantasy, Monaco',
    'themeVariables': {
      'primaryColor': '#2FB1',
      'primaryTextColor': '#F8B229',
      'lineColor': '#F8B229',
      'primaryBorderColor': '#27AE60',
      'secondaryColor': '#EBDEF0',
      'secondaryTextColor': '#6C3483',
      'secondaryBorderColor': '#A569BD',
      'fontSize': '20px'
    }
  }
}%%
flowchart TD
    EStart["Error Occurs"]
    EStart --> EPerm["Permission Denied"]
    EStart --> ESpeechRec["Speech Recognition Error"]
    EStart --> EAudio["Microphone/Audio Error"]
    EStart --> EAIInit["AI Initialization Failure"]
    EStart --> EAICall["AI Processing Error"]
    EStart --> EPlayback["TTS/Audio Output Error"]

    EPerm --> UIError["Show permission error (red)"]
    ESpeechRec --> UIError
    EAudio --> UIError
    EAIInit --> UIError
    EAICall --> UIError
    EPlayback --> UIError

    UIError["Display error message on card and reset state"]
```

---

### G. Core Data Structures & Dependencies

```mermaid
---
title: "Core Data Structures & Dependencies"
author: "Cong Le"
version: "1.0"
license(s): "MIT, CC BY 4.0"
copyright: "Copyright (c) 2025 Cong Le. All Rights Reserved."
config:
  look: handDrawn
  theme: base
---
%%%%%%%% Mermaid version v11.4.1-b.14
%%{
  init: {
    'classDiagram': { 'htmlLabels': false},
    'fontFamily': 'Fantasy, Monaco',
    'themeVariables': {
      'primaryColor': '#B259',
      'primaryTextColor': '#F8B229',
      'lineColor': '#F8B229',
      'primaryBorderColor': '#7C2',
      'secondaryColor': '#22F',
      'fontSize': '20px'
    }
  }
}%%
classDiagram
    class AICardView {
        +aiResponse: String?
        +userQuestion: String?
        +isLoading: Bool
        +errorMessage: String?
        +isRecording: Bool
        +speechRecognizer: SFSpeechRecognizer
        +recognitionRequest: SFSpeechAudioBufferRecognitionRequest?
        +recognitionTask: SFSpeechRecognitionTask?
        -audioEngine: AVAudioEngine
        -speechSynthesizer: AVSpeechSynthesizer
        +runDemoAIModel(question): String
        +fetchAIResponse(question)
        +speak(text)
        +requestSpeechAuthorization()
        +startRecording()
        +stopRecording()
        +toggleRecording()
    }
    class AIError {
        <<enum>>
        initializationFailed
        processingError
        speechRecognitionError
        permissionDenied
    }
    class LLM
    class SFSpeechRecognizer
    class AVAudioEngine
    class AVSpeechSynthesizer

    AICardView --> LLM : uses for AI
    AICardView --> SFSpeechRecognizer : speech-to-text
    AICardView --> AVAudioEngine : audio input
    AICardView --> AVSpeechSynthesizer : text-to-speech
    AICardView --> AIError
```

---

### H. Permissions Decision Tree

```mermaid
---
title: "Permissions Decision Tree"
author: "Cong Le"
version: "1.0"
license(s): "MIT, CC BY 4.0"
copyright: "Copyright (c) 2025 Cong Le. All Rights Reserved."
config:
  layout: dagre
  theme: base
---
%%%%%%%% Mermaid version v11.4.1-b.14
%%%%%%%% Available curve styles include the following keywords:
%% basis, bumpX, bumpY, cardinal, catmullRom, linear, monotoneX, monotoneY, natural, step, stepAfter, stepBefore.
%%{
  init: {
    'flowchart': { 'htmlLabels': false, 'curve': 'linear' },
    'fontFamily': 'Fantasy, Monaco',
    'themeVariables': {
      'primaryColor': '#2FB1',
      'primaryTextColor': '#F8B229',
      'lineColor': '#F8B229',
      'primaryBorderColor': '#27AE60',
      'secondaryColor': '#EBF2',
      'secondaryTextColor': '#6C3483',
      'secondaryBorderColor': '#A569BD',
      'fontSize': '20px'
    }
  }
}%%
flowchart TD
    StartPerm["App launches or mic tapped"]
    StartPerm --> SRPerm{"Speech recognition permission status?"}
    SRPerm -- Authorized --> MicPerm
    SRPerm -- "Denied/<br/>Restricted/<br/>Unknown" --> FailSR
    MicPerm{"Microphone permission status?"}
    MicPerm -- Granted --> Proceed["Proceed to record"]
    MicPerm -- Denied --> FailMic
    FailSR["Show:<br/>Cần cấp quyền nhận dạng<br/>giọng nói và micro"]
    FailMic["Show:<br/>Cần cấp quyền micro"]
```

---

### I. Data & UI State Synchronization

```mermaid
---
title: "Data & UI State Synchronization"
author: "Cong Le"
version: "1.0"
license(s): "MIT, CC BY 4.0"
copyright: "Copyright (c) 2025 Cong Le. All Rights Reserved."
config:
  layout: dagre
  theme: base
---
%%%%%%%% Mermaid version v11.4.1-b.14
%%%%%%%% Available curve styles include the following keywords:
%% basis, bumpX, bumpY, cardinal, catmullRom, linear, monotoneX, monotoneY, natural, step, stepAfter, stepBefore.
%%{
  init: {
    'flowchart': { 'htmlLabels': false, 'curve': 'linear' },
    'fontFamily': 'Fantasy, Monaco',
    'themeVariables': {
      'primaryColor': '#2FB1',
      'primaryTextColor': '#F8B229',
      'lineColor': '#F8B229',
      'primaryBorderColor': '#27AE60',
      'secondaryColor': '#DEF2',
      'secondaryTextColor': '#6C3483',
      'secondaryBorderColor': '#A569BD',
      'fontSize': '20px'
    }
  }
}%%
flowchart TB
    UserQuestion["userQuestion"]
    AIResponse["aiResponse"]
    IsLoading["isLoading"]
    ErrorMsg["errorMessage"]
    IsRecording["isRecording"]

    UserQuestion -.->|Displayed in UI| UI
    AIResponse -.->|Displayed & spoken| UI
    IsLoading -.->|Show spinner/disable| UI
    ErrorMsg -.->|Show error message| UI
    IsRecording -.->|Update button color & text| UI
```

---

### J. Quantization Model Impact (AI Loading)

```mermaid
---
title: "Quantization Model Impact (AI Loading)"
author: "Cong Le"
version: "1.0"
license(s): "MIT, CC BY 4.0"
copyright: "Copyright (c) 2025 Cong Le. All Rights Reserved."
config:
  layout: dagre
  theme: base
---
%%%%%%%% Mermaid version v11.4.1-b.14
%%%%%%%% Available curve styles include the following keywords:
%% basis, bumpX, bumpY, cardinal, catmullRom, linear, monotoneX, monotoneY, natural, step, stepAfter, stepBefore.
%%{
  init: {
    'flowchart': { 'htmlLabels': false, 'curve': 'linear' },
    'fontFamily': 'Fantasy, Monaco',
    'themeVariables': {
      'primaryColor': '#2FB1',
      'primaryTextColor': '#F8B229',
      'lineColor': '#F8B229',
      'primaryBorderColor': '#27AE60',
      'secondaryColor': '#EEF2',
      'secondaryTextColor': '#6C3483',
      'secondaryBorderColor': '#A569BD',
      'fontSize': '20px'
    }
  }
}%%
flowchart TD
    ModelChoice["LLM Model Quantization"]
    Q8("Q8_0<br/>(8-bit):<br/>High quality, more memory")
    Q5("Q5_K_M<br/>(5-bit):<br/>Middle ground")
    Q4("Q4_K_M<br/>(4-bit):<br/>Low memory, good quality")
    Q3("Q3_K_M<br/>(3-bit):<br/>Smallest, may degrade quality")

    ModelChoice --> Q8
    ModelChoice --> Q5
    ModelChoice --> Q4
    ModelChoice --> Q3
    Q4 -->|selected| ModelLoading
    Q8 & Q5 & Q3 -->|possible alternative| ModelLoading
```
*Shows developer’s comments on memory/quality tradeoffs.*

---

## 3. Concept Map: End-to-End User Journey

```mermaid
---
title: "Concept Map: End-to-End User Journey"
author: "Cong Le"
version: "1.0"
license(s): "MIT, CC BY 4.0"
copyright: "Copyright (c) 2025 Cong Le. All Rights Reserved."
config:
  look: handDrawn
  theme: base
---
%%%%%%%% Mermaid version v11.4.1-b.14
%%{
  init: {
    'journey': { 'htmlLabels': false},
    'fontFamily': 'Fantasy, Monaco',
    'themeVariables': {
      'primaryColor': '#BB28',
      'primaryTextColor': '#F8B229',
      'lineColor': '#F8B229',
      'primaryBorderColor': '#7C3',
      'secondaryColor': '#DD15',
      'fontSize': '20px'
    }
  }
}%%
journey
    title User Journey with AI Vietnamese Chat Card
    section Grant Permissions
      Open app: 5: App
      Grant permissions: 3: User/OS
    section Ask by Voice
      Tap mic & speak: 5: User
      Live transcription: 4: UI
      Stop recording (auto or tap): 5: User/UI
    section AI Processing
      Question sent to LLM: 4: UI
      "Đang xử lý..." spinner: 4: UI
      Receive answer: 5: LLM
    section Output
      Display answer, speak aloud: 5: UI/TTS
      Ready for next question: 5: UI
    section Error Path
      Permission/processing error: 1: UI
      Display clear error msg: 1: UI
```

---

# Summary Table

| Aspect                         | Visual Diagram | Concepts Illustrated                                |
|---------------------------------|---------------|-----------------------------------------------------|
| **Overall Data Flow**           | A             | All moving parts: UI, AI, speech, permissions       |
| **UI State Transitions**        | B             | States: idle, recording, processing, error, answer  |
| **Interaction Sequence**        | C             | Timeline and message sequence for a user action     |
| **AI Model Handling**           | D             | Model initialization, error branches                |
| **Speech Recognition Details**  | E             | Stepwise logic from tap to transcription & error    |
| **Error Handling**              | F             | Mapping errors to user-facing UI                    |
| **Class & State Structure**     | G             | Objects, state variables, dependencies              |
| **Permissions Logic**           | H             | Nested permission checks and outcomes               |
| **State Synchronization**       | I             | UI updates and their triggers                       |
| **LLM Quantization Choice**     | J             | Memory/quality tradeoffs in LLM selection           |
| **User Journey**                | Journey Map   | Bird’s-eye view of app usage scenario               |

---

## How to Read These Diagrams

- **Flowcharts:** Show logical or data pathways and choices.
- **State diagrams:** Depict how the UI state changes with actions or results.
- **Sequence diagrams:** Clarify interaction timeline and message flow between components and actors.
- **Class diagrams:** Map out how code entities connect.
- **Journey diagrams:** Visualize the high-level user-centered journey.
- **Quantization tradeoff:** Relates a developer concern.

---

## Final Thoughts

This implementation is a rich, modern example of bridging **speech, AI, stateful UI, and error handling** in SwiftUI. The diagrams collectively capture not just "how" the individual parts work, but **how the entire user experience is woven together**—from permissions and state management, through asynchronous voice and AI handling, to final user output and interactivity. 


---

*These diagrams can be embedded in your documentation, onboarding materials, or code comments to make codebases like this approachable for teams and stakeholders alike.*


---


<!-- 
```mermaid
%% Current Mermaid version
info
```  -->


```mermaid
---
title: "CongLeSolutionX"
author: "Cong Le"
version: "1.0"
license(s): "MIT, CC BY 4.0"
copyright: "Copyright (c) 2025 Cong Le. All Rights Reserved."
config:
  theme: base
---
%%%%%%%% Mermaid version v11.4.1-b.14
%%{
  init: {
    'flowchart': { 'htmlLabels': false },
    'fontFamily': 'Bradley Hand',
    'themeVariables': {
      'primaryColor': '#fc82',
      'primaryTextColor': '#F8B229',
      'primaryBorderColor': '#27AE60',
      'secondaryColor': '#81c784',
      'secondaryTextColor': '#6C3483',
      'lineColor': '#F8B229',
      'fontSize': '15px'
    }
  }
}%%
flowchart LR
    My_Meme@{ img: "https://raw.githubusercontent.com/CongLeSolutionX/MY_GRAPHIC_ASSETS/refs/heads/Designing_graphic_syntax/MY_MEME/My-meme-icon-design.png", label: "Ăn uống gì chưa ngừi đẹp?", pos: "b", w: 200, h: 150, constraint: "on" }

    Closing_quote@{ shape: braces, label: "I'll leave this Earth empty-handed anyway!<br/>YOLO" }

My_Meme ~~~ Closing_quote

```


---
>**Licenses:**
>
>- **MIT License:**  [![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE) - Full text in [LICENSE](LICENSE) file.
>- **Creative Commons Attribution 4.0 International:** [![License: CC BY 4.0](https://licensebuttons.net/l/by/4.0/88x31.png)](LICENSE-CC-BY) - Legal details in [LICENSE-CC-BY](LICENSE-CC-BY) and at [Creative Commons official site](http://creativecommons.org/licenses/by/4.0/).