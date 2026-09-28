## genui-codelab

### Intro to GenUI with Flutter — DevFest 2026 Codelab

A ready-to-use development environment for the [Intro to GenUI](https://codelabs.developers.google.com/codelabs/genui-intro#0) codelab. Opens directly in GitHub Codespaces — no local Flutter installation required.

[![Open in GitHub Codespaces](https://github.com/codespaces/badge.svg)](https://codespaces.new/Kymoraa/genui-codelab)

---
#### What you'll build

An AI-powered task management app using Flutter and the GenUI SDK. A Firebase AI Logic agent handles natural language input and generates real Flutter UI components at runtime — no hardcoded layouts.

---
#### Before you start

You'll need:

- A GitHub account — used to sign into Codespaces (free, 60 hrs/month included)
- A Google account — used to create a Firebase project
- A Firebase project — you'll create this during the codelab (step 3)
- No prior GenUI or MCP knowledge required
- Intermediate Flutter experience is helpful but not essential

> [!IMPORTANT]
> Each attendee must use their **own** Firebase project — do not share one.
> Sharing a project will cause rate limit errors (429) for everyone.

---
#### Using the Codespaces environment

1. Click **Open in GitHub Codespaces** above and sign in with your GitHub account
2. Codespaces will build the container and set up Flutter, Dart, and the Firebase CLI — this takes about 2 minutes on first launch
3. When the workspace opens, open the terminal and follow the codelab from **Step 2: Create the Flutter project**
4. Run the app with:

`flutter run -d web-server --web-port 8080 --web-hostname 0.0.0.0 --web-renderer html` This command will avoid a known Flutter CanvasKit console error in Codespaces. Otherwise you can also run: `flutter run -d web-server --web-port 8080 --web-hostname 0.0.0.0` 

5. Go to the Ports tab → right-click port 8080 → set Port Visibility to Public → open the URL via the globe icon.
6. The first load in debug mode takes 1–3 minutes while Flutter compiles — the page will appear blank. This is normal. Wait, then refresh if needed.

Hot reload and hot restart are available in the terminal:
- r — hot reload (preserves state)
- R — hot restart (clears state)

The environment includes:
- Flutter >= 3.35.7 (stable)
- Dart >= 3.9
- Firebase CLI
- Flutter and Dart VS Code extensions

---
#### Firebase CLI

If firebase is not found, install it manually:
`curl -sL https://firebase.tools | bash`

Firebase authentication in Codespaces

Codespaces is a headless environment, so the standard firebase login flow won't work. Use the CI token flow instead:

1. Generate a token:
firebase login:ci --no-localhost
2. Open the URL it prints in your local browser and sign in with your Google account
3. Copy the authorization code shown on screen and paste it back into the Codespaces terminal
4. Firebase will print a long token — export it and save it to ~/.bashrc so it persists across sessions:
`export FIREBASE_TOKEN=<paste-token-here>`
`echo 'export FIREBASE_TOKEN=<paste-token-here>' >> ~/.bashrc`
`echo 'export PATH="$PATH":"$HOME/.pub-cache/bin"' >> ~/.bashrc`

---
#### FlutterFire CLI

When the codelab asks you to run `dart pub global activate flutterfire_cli`, the flutterfire command won't be found immediately. Run this first:

`export PATH="$PATH":"$HOME/.pub-cache/bin"`

---
#### App Check
> [!IMPORTANT]
> This step is required for the app to work. Skipping it means the agent will
> never respond, with no visible error in the app.

By default, Firebase AI Logic has App Check enforced on every new Firebase project. Switch it to Monitor mode:

1. Go to Firebase Console (https://console.firebase.google.com) → your project
2. Build → App Check → APIs tab
3. Find Firebase AI Logic → click Show details
4. Click Set up → switch enforcement to Monitor

---
#### Rate limits

If you see no response from the agent and the browser console shows a 429 error:

- You have exceeded the free tier quota (20 requests/minute on Gemini)
- Wait 60 seconds and try again
- Or switch the Gemini model. E.g., `gemini-2.5-flash`, `gemini-3.5-flash`, `gemini-3.5-flash-lite` etc.

---
#### Resuming your session

Codespaces suspends after 30 minutes of inactivity. Your files and installed tools are preserved, but environment variables and running processes are not.

When you reconnect:

1. Re-load environment variables: `source ~/.bashrc` 
Otherwise re-export manually:
`export FIREBASE_TOKEN=<your-token>`
`export PATH="$PATH":"$HOME/.pub-cache/bin"`
2. Set port 8080 back to Public in the Ports tab
3. Restart the app: `flutter run -d web-server --web-port 8080 --web-hostname 0.0.0.0 --web-renderer html` or `flutter run -d web-server --web-port 8080 --web-hostname 0.0.0.0`

Again, Expect the 1-3 minute wait time as the Codespace loads

---
#### Running locally

If you have the Flutter SDK installed and prefer to work locally:

Requirements
- Flutter >= 3.35.7 (run flutter upgrade if needed)
- Dart >= 3.9 (ships with Flutter 3.35)
- Firebase CLI — install with npm install -g firebase-tools

Then follow the codelab from Step 1.

---
#### Known issues

`LateInitializationError: Field '_handledContextLostEvent'` in the console

This is a Flutter web engine bug with the CanvasKit (WebGL) renderer — not your code. It appears when the browser briefly loses the GPU context, which can happen in Codespaces. The app continues to work normally and you can safely ignore it. Using `--web-renderer html` (included in the run command above) avoids this entirely.

---
#### Resources

- Codelab: Intro to GenUI with Flutter (https://codelabs.developers.google.com/codelabs/genui-intro#0)
- GenUI package on pub.dev (https://pub.dev/packages/genui)
- Firebase AI Logic docs (https://firebase.google.com/docs/ai-logic)
- GitHub Codespaces docs (https://docs.github.com/en/codespaces)
