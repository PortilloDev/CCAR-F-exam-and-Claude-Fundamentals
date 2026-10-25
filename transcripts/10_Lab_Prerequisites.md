# Lecture 10: Lab: Prerequisites

**Section:** Introduction to the CCAR-F exam and Claude Fundamentals  
**Duration:** 8 min | **Lecture ID:** 58217323

---

## Transcript

Welcome, so we are now preparing you so that you have all the prerequisites necessary so that you can run the labs inside your own laptop and environment. Let's get started.

**Step 1: Download the code repository**
Follow the link attached in the lecture. Click on Code → Download ZIP, or copy the SSH link and run `git clone <url>` in your terminal. If you don't have git CLI, follow the attached link to install git.

**Step 2: Install Homebrew (Mac)**
Install Homebrew — the easiest way to manage dependencies on Mac. After installing Homebrew, install a Python environment using it.

**Step 3: Install uv as package manager**
Use `uv` — the fastest way to install and cache Python libraries. Install via Homebrew. For Linux/Windows users, a document with additional instructions is included in the course resources.

**Step 4: Install an IDE**
Use Visual Studio Code (free) or a similar derivative like the one shown (anti-gravity IDE — same look and feel as VS Code).

**Step 5: Open the project and install the Jupyter extension**
Open VS Code, go to Extensions, search for "Jupyter", install it. This is required since the labs use Jupyter Notebooks — an easy way to run Python code in cells.

Open the project folder. Open the terminal inside VS Code and run:
```
uv sync
```
This initializes a virtual environment in `.venv/` to isolate project dependencies.

**Step 6: Configure the kernel**
When opening a notebook, click Kernel → Python Environments and select the uv-created environment. Repeat for every notebook.

**Step 7: Configure environment variables (.env)**
The project includes a `.env.example` file that defines two model names:
- `claude-haiku-4-5` (used for most labs — cheapest model)
- A second model

Duplicate `.env.example`, rename to `.env`, remove the `.example` part, keep the two model names. The labs mainly use claude-haiku-4-5 to minimize cost.

**Step 8: Log into Claude Code**
If you have an Anthropic subscription, install Claude Code following the instructions in the lecture. Then run:
```
claude
```
Trust the folder and log in if prompted. Verify by asking Claude something like "What is this project about?"

**API Key (optional, required for Lab 5)**
If using API key pricing instead of subscription: go to Credits, turn off auto-reload to control your budget (minimum charge is $5 + tax). Create an API key in the console, copy it, insert into `.env` (commented out initially). Uncomment only when running Lab 5, then re-comment afterwards.

**Summary of prerequisites:**
1. Code repository cloned ✓
2. Homebrew installed (Mac) ✓
3. Python + uv installed ✓
4. VS Code + Jupyter extension installed ✓
5. Virtual environment initialized with `uv sync` ✓
6. Claude Code logged in ✓
7. `.env` file configured ✓
