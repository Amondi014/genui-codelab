# Intro to GenUI with Flutter — Comprehensive Codelab Walkthrough

> **Google Codelabs Reference:** [Build a Generative UI (GenUI) App](https://codelabs.developers.google.com/codelabs/genui-intro)  
> **Repository:** [Amondi014/genui-codelab](https://github.com/Amondi014/genui-codelab)

---

## 1. What is Generative UI (GenUI)?

In traditional mobile and web development, user interfaces are **statically declared** by developers ahead of time. User journeys and screens must be pre-planned, hardcoded, and compiled.

**Generative UI (GenUI)** shifts this paradigm:
- An AI model (such as Google Gemini via Firebase AI Logic) dynamically decides **what UI components to present** and **how to configure them** based on natural language dialogue and the user's immediate intent.
- Instead of returning raw unstructured text or fragile markdown, the AI agent outputs structured messages conforming to the **A2UI (Agent-to-UI)** protocol.
- The Flutter client maps these A2UI payloads against a predefined **Catalog** of verified Flutter widgets (such as task lists, forms, interactive buttons, or progress bars).
- When a user interacts with a generated component (e.g., ticking a checkbox), the component emits a **UserActionEvent** back to the AI model, allowing bidirectional conversational workflows with rich UI state.

```
       +-------------------------------------------------------------+
       |                         User                                |
       +-------------------------------------------------------------+
                        |                          ^
           Natural      |                          | Rich Flutter
          Language      v                          | Widget Surfaces
       +-------------------------------------------------------------+
       |               Flutter App (GenUI SDK)                       |
       |  - SurfaceController                                        |
       |  - Catalog (BasicCatalog + Custom TaskDisplay)              |
       |  - A2uiTransportAdapter                                     |
       +-------------------------------------------------------------+
                        |                          ^
              A2UI      |                          | A2UI
           Events /     v                          | Component JSON
       +-------------------------------------------------------------+
       |               Firebase AI Logic (Gemini)                    |
       |  - System Instruction & Schema Catalog                      |
       |  - Multi-Turn Conversation Session                          |
       +-------------------------------------------------------------+
```

---

## 2. Project Architecture & Components

The application is structured into the following key building blocks:

| File | Purpose |
| :--- | :--- |
| `pubspec.yaml` | Declares dependencies: `genui`, `firebase_ai`, `firebase_core`, `json_schema_builder`. |
| `lib/main.dart` | Main entry point, Firebase initialization, `SurfaceController`, `A2uiTransportAdapter`, and `Conversation` orchestration. |
| `lib/firebase_options.dart` | Firebase configuration for web, Android, iOS, macOS, and Windows. |
| `lib/message_bubble.dart` | Chat message bubble widget with modern gradients and responsive alignment. |
| `lib/task_display.dart` | Custom GenUI `CatalogItem` defining the schema, data model, and interactive `CheckboxListTile` column for task management. |
| `steps/` | Milestone snapshots (`step_03` through `step_07`) representing each stage of the codelab. |

---

## 3. Step-by-Step Codelab Progression

### Step 1: Overview & Prerequisites
- Understand declarative vs. generative UI concepts.
- Requirements: Flutter SDK (>= 3.35.7), Dart (>= 3.9), Node.js, and Firebase CLI (`npm install -g firebase-tools`).

### Step 2: Project Creation & Dependencies
Create the Flutter project and add required packages to `pubspec.yaml`:
```yaml
dependencies:
  flutter:
    sdk: flutter
  firebase_core: ^4.14.0
  firebase_ai: ^4.0.0
  genui: ^0.10.3
  json_schema_builder: ^0.1.7

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^6.0.0
```

### Step 3: Firebase Project & App Check Setup
1. **Initialize Firebase in Flutter:**
   ```dart
   WidgetsFlutterBinding.ensureInitialized();
   await Firebase.initializeApp(
     options: DefaultFirebaseOptions.currentPlatform,
   );
   ```
2. **App Check Configuration (Critical):**
   - In the [Firebase Console](https://console.firebase.google.com), open **Build** -> **App Check** -> **APIs**.
   - Under **Firebase AI Logic**, click **Show details** -> **Set up** -> switch enforcement to **Monitor mode**.

### Step 4: Conversational Chat Interface
- Build a standard chat interface with `MessageBubble` widgets, a `TextField`, and a `ListView`.
- Initialize `FirebaseAI.googleAI().generativeModel(model: 'gemini-3.5-flash')` and start a chat session (`_chatSession = model.startChat()`).

### Step 5: Integrating the GenUI SDK
- Initialize the GenUI catalog with built-in primitive components:
  ```dart
  catalog = BasicCatalogItems.asCatalog();
  _controller = SurfaceController(catalogs: [catalog]);
  _transport = A2uiTransportAdapter(onSend: _sendAndReceive);
  _conversation = Conversation(
    controller: _controller,
    transport: _transport,
  );
  ```
- Listen for `_conversation.events` (`ConversationSurfaceAdded`, `ConversationSurfaceRemoved`, `ConversationContentReceived`).
- Embed a `Surface` widget in your widget tree bound to the surface ID:
  ```dart
  Surface(
    surfaceContext: _controller.contextFor(taskDisplaySurfaceId),
  )
  ```

### Step 6: AI-Driven Surface Generation
Configure the Gemini agent with system instructions and inject the catalog schema into the system prompt:
```dart
final promptBuilder = PromptBuilder.chat(
  catalog: catalog,
  systemPromptFragments: [systemInstruction],
);

_conversation.sendRequest(
  ChatMessage.system(promptBuilder.systemPromptJoined()),
);
```

The system prompt directs the agent to act as an expert task planner, ask clarifying questions, assemble the task list, and render it into the `task_display` surface ID using the A2UI format.

### Step 7: Custom GenUI Component (`TaskDisplay`)
Define a customized schema and widget builder in `lib/task_display.dart`:

```dart
final taskDisplaySchema = S.object(
  properties: {
    'component': S.string(enumValues: ['TaskDisplay']),
    'title': S.string(description: 'The title of the task list'),
    'tasks': S.list(
      description: 'A list of tasks to be completed today',
      items: S.object(
        properties: {
          'name': S.string(description: 'The name of the task to be completed'),
          'isCompleted': S.boolean(description: 'Whether the task is completed'),
          'completeAction': A2uiSchemas.action(
            description: 'The action performed when the user has completed the task.',
          ),
        },
        required: ['name', 'isCompleted', 'completeAction'],
      ),
    ),
  },
  required: ['title', 'tasks'],
);
```

When a user taps the checkbox in the UI, an event is dispatched directly back to the AI agent:
```dart
itemContext.dispatchEvent(
  UserActionEvent(
    name: task.actionName,
    sourceComponentId: itemContext.id,
    context: resolvedContext,
  ),
);
```

The catalog in `lib/main.dart` is updated to include `taskDisplay`:
```dart
catalog = BasicCatalogItems.asCatalog().copyWith(newItems: [taskDisplay]);
```

---

## 4. How to Run the Application

### Option A: GitHub Codespaces (Zero Setup)
1. Navigate to [github.com/Amondi014/genui-codelab](https://github.com/Amondi014/genui-codelab).
2. Click **Code** -> **Codespaces** -> **Create codespace on main**.
3. Codespaces automatically runs `.devcontainer/devcontainer.json` installing Flutter, Dart, and Firebase CLI.
4. Launch the web application:
   ```bash
   flutter run -d web-server --web-port 8080 --web-hostname 0.0.0.0 --web-renderer html
   ```
5. In the **Ports** tab, right click port 8080 and select **Port Visibility: Public**, then click the globe icon to open the app.

### Option B: Local Flutter Installation
1. Ensure Flutter SDK (>= 3.35.7) is on your PATH.
2. Authenticate Firebase:
   ```bash
   firebase login
   dart pub global activate flutterfire_cli
   flutterfire configure
   ```
3. Fetch dependencies and run:
   ```bash
   flutter pub get
   flutter run -d chrome
   ```

---

## 5. Troubleshooting & Tips

- **429 Rate Limits:** The free Gemini tier permits 20 requests per minute. If you hit this limit, wait 60 seconds or switch the model in `lib/main.dart` (e.g. `gemini-2.5-flash` or `gemini-3.5-flash-lite`).
- **CanvasKit WebGL Warning:** Use `--web-renderer html` when running in browser containers to ensure compatibility across headless GPU environments.
- **Agent Doesn't Respond:** Confirm that Firebase App Check has Firebase AI Logic in **Monitor mode**.
