# Duck Hours

Duck Hours is an AI-assisted coding and debugging workspace for students. It combines a browser-based code editor, local program execution, guided hints, and generated test cases to help students understand errors and investigate code that runs but produces the wrong result.

The project uses a hint-first workflow: after a failed run, students can request a targeted hint before choosing an AI-generated fix. Students can also describe their program's purpose and constraints to help the logic checker evaluate its behavior.

## Features

- **Multi-language editor:** Write Python, JavaScript, TypeScript, Ruby, Go, Java, C, and C++ using the Monaco editor.
- **Local code execution:** Run code with the compiler or interpreter installed on the server machine and inspect its output.
- **Guided debugging:** Request one specific line hint at a time, with an optional fix available after requesting a hint.
- **Logic checks:** Review successful runs for likely mistakes against the stated purpose, constraints, and observed output.
- **Generated test cases:** Highlight a complete function or method and generate 1–20 cases with expected results and separate test harnesses.
- **Browser-local workspaces:** Keep separate files, settings, and test cases in local storage.
- **Error explanations:** Follow beginner-friendly guides for highlighted errors.

## Example workflow

1. Open a workspace and write or paste your code into the editor.
2. Describe what the program should do and any relevant constraints.
3. Run the program and inspect its output or error.
4. If it fails, choose **Give me a hint** and use the feedback to investigate. **Fix it for me** becomes available after a hint.
5. To test a particular function or method, highlight its complete definition and use the **Testcases** panel.
6. Compare actual results with expected behavior, revise the code, and run it again.

## Technology stack

| Layer | Technology and role |
| --- | --- |
| Frontend | HTML, CSS, JavaScript, and Monaco for the browser workspace |
| Backend | Python and FastAPI for HTTP endpoints and application logic |
| Development server | Uvicorn |
| AI integration | Meta API for analysis, hints, fixes, and test generation |
| Execution | Local language runtimes and compilers |
| Workspace storage | Browser local storage |

FastAPI serves the web interface and API routes. The browser sends execution requests to the local runner and AI requests to the backend. Running code does not require an AI API key; AI features require configuration.

## Project status

Duck Hours is a local-development prototype. AI feedback and generated expected results should be reviewed; they do not prove that a program is correct. Workspaces are stored in the current browser rather than a shared account.

## Code layout

- `app/schemas.py` defines API request and response models.
- `app/analysis.py` handles demo analysis, AI hints, and testcase generation.
- `app/runner.py` handles local code execution and generated files.
- `app/test_targets.py` detects programs, functions, methods, and safe selected fragments and builds separate test wrappers.
- `app/routes.py` defines the HTTP endpoints. `app/api.py` keeps the existing router import available.
- `static/editor-common.js`, `static/workspaces.js`, `static/editor.js`, `static/runner.js`, `static/testcases.js`, and `static/editor-init.js` load in that order and separate shared state, browser-local workspaces, file tabs, execution/hints, testcases, and Monaco startup.
- `static/workspace.css` contains workspace-specific styles; `static/styles.css` contains the shared base styles.

## Run locally

You need a Meta API key for AI features; see example.ev to add a private key.

From the repository directory:

```bash
python -m venv .venv
# Windows PowerShell: .\.venv\Scripts\Activate.ps1
# macOS/Linux: source .venv/bin/activate
python -m pip install -r requirements.txt
python -m uvicorn app.main:app --reload
```

If `python3` is not installed on macOS, install Python 3.9+ from python.org or Homebrew first. Then open http://127.0.0.1:8000.

On Windows, you can also run `.\start-server.ps1` from PowerShell. Then open http://127.0.0.1:8000. Running code works without an API key. To enable AI hints, copy `.env.example` to `.env`, add your `META_API_KEY`, and restart the server. The `.env` file is ignored by Git.

The browser editor supports Python, JavaScript, TypeScript, Ruby, Go, Java, C, and C++. The Run button uses the local installed compiler/interpreter and does not call AI. Successful programs show their output. After every successful run, Meta checks the program purpose, constraints, and observed output for likely logic issues; the `Skip the logic check this run` checkbox provides an explicit bypass. A failed run stays neutral until the student chooses `Give me a hint`; only then does `Fix it for me` become available. Meta returns one specific line hint at a time. A standalone Python function is run behind the scenes with small automatic test inputs. Error guide links explain the highlighted error in beginner-friendly language.

The Testcases panel tests only a highlighted, complete function or method, including a selected `main` function or method. Choose 1–20 cases (default 5), then generate tests; the panel shows calls, expected return values or printed output, separate generated harnesses, results, and when results are outdated. Use the normal Run button for a whole program. A Java method without `main` shows the notice `Input main method.` and remains testable from the Testcases panel.

The Workspaces button keeps separate files, settings, and test cases in this browser's local storage. Closing a file tab keeps it in the Files panel; deleting it there removes it from that workspace. AI may return seeded-random or boundary-test recipes alongside concrete cases. Recipes are shown for inspection and are never executed as code. Required-file checks compare the post-run workspace snapshot, including files that already existed. Optional memory limits use sampled process-tree usage; the runner may report that memory could not be measured. Running submitted code still requires a trusted local environment: the temporary runner is resource-limited but is not a hardened sandbox.
