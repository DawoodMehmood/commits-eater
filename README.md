# Commits Eater
Generate a snake eating commits graph game from your own GitHub contributions graph.
This mini Github workflow will grab your contribution graph and generate this interesting devouring visualization for both light and dark mode:

<picture>
  <source
    media="(prefers-color-scheme: dark)"
    srcset="example/github-contribution-grid-snake-dark.svg"
  />
  <source
    media="(prefers-color-scheme: light)"
    srcset="example/github-contribution-grid-snake.svg"
  />
  <img alt="Snake Game" src="images/github-contribution-grid-snake.svg" />
</picture>

## 1. Copy the Workflow

You don’t need to fork the whole repo. Just create a GitHub Actions workflow in your own **profile repository** by following these steps:

* Go to your profile repo (`github.com/<your-username>/<your-username>`) or create one if you havent already.
* Create a folder: `.github/workflows/`
* Inside it, add a file like `snake.yml` with this exact content, no additional changes needed:

```yaml
name: generate contribution snake

on:
  schedule:
    - cron: '0 0 * * *'
  workflow_dispatch:

jobs:
  generate:
    permissions:
      contents: write   # allows the job to push generated files back to the repo
    runs-on: ubuntu-latest
    timeout-minutes: 10

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          persist-credentials: true  # keep credentials so git push uses the provided token

      - name: Generate contribution snake SVGs
        uses: Platane/snk/svg-only@v3
        with:
          github_user_name: ${{ github.repository_owner }}
          outputs: |
            dist/github-contribution-grid-snake.svg
            dist/github-contribution-grid-snake-dark.svg?palette=github-dark

      - name: Move generated SVGs into images/
        run: |
          mkdir -p images
          mv dist/github-contribution-grid-snake.svg images/github-contribution-grid-snake.svg || true
          mv "dist/github-contribution-grid-snake-dark.svg" images/github-contribution-grid-snake-dark.svg || true

      - name: Configure git
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"

      - name: Commit and push SVGs
        run: |
          git add images/github-contribution-grid-snake.svg images/github-contribution-grid-snake-dark.svg || true
          git commit -m "chore: update snake SVGs" || echo "No changes to commit"
          git push
```

This action will automatically fetch your contribution graph and regenerate the snake game SVGs daily.

---

## 2. Update Your Profile README

In the `README.md` of your profile repo (`github.com/<your-username>/<your-username>`), add this snippet:

```html
<picture>
  <source
    media="(prefers-color-scheme: dark)"
    srcset="images/github-contribution-grid-snake-dark.svg"
  />
  <source
    media="(prefers-color-scheme: light)"
    srcset="images/github-contribution-grid-snake.svg"
  />
  <img alt="Snake Game" src="images/github-contribution-grid-snake.svg" />
</picture>
```

This way, your profile will show the light or dark snake game depending on visitors’ GitHub theme.

---

## 3. Push & Wait

* Commit and push the workflow + README changes.
* GitHub Actions will run, generate the images, and commit them into your repo under `/images/`.
* Your profile README will then start showing the game 🎉

--------------------------------------------------------------------------------------------------------

## Run Instantly

When you commit the workflow for the first time, you will see the output after 24 hours due to the set cron job.
But you don’t need to wait, you can trigger it manually:

1. Go to your same profile repo (`github.com/<your-username>/<your-username>`).
2. Click **Actions** tab.
3. Select your workflow (likely called *“generate contribution snake”*).
4. On the right, there’s a **“Run workflow”** button (because we added `workflow_dispatch:` in your yml).
5. Click it → wait for a few seconds and it will run the job and generate the game after a minute or so.
6. Go to your profile page and you will see the game.

After the workflow finishes:

* Refresh your repo → you should see the new `images/` folder with the SVGs.
* Your profile README will then render the snake game automatically.

⚡ Tip: if you don’t see the **Run workflow** button, make sure Actions are enabled for your repo (`Settings → Actions → General → Allow all actions`).


## Change Frequency

If you copy paste the exact workflow given above, it will run every 24 hrs due to the set cron job:

```yaml
on:
  schedule:
    - cron: "0 0 * * *"
```

If you want the game to be updated on your every commit then replace the above cron job with this:

```yaml
name: generate contribution snake

on:
  push:
      branches:
        - main   # or "master" depending on your default branch
```
### Note

No need to worry about ever increasing storage because the way that workflow is written:

* It **always overwrites** the files `images/github-contribution-grid-snake.svg` and `images/github-contribution-grid-snake-dark.svg`.
* So your repo will only ever have **two files**, one for light and one for dark mode.
* Older versions aren’t kept — they’re replaced on every run.


