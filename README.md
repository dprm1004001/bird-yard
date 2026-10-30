# 새마당 (Bird Yard)

In a blank white yard, you shake a berry tree and trivial little low-poly birds come to eat the fallen fruit.
Collect the feathers they drop, open new places and fill the bird encyclopedia.

It is a single-page web game built from one HTML file (`index.html`) and Three.js (r128, loaded from a CDN). There is no build step.

## 플레이

- Tap a tree, basket or snack bag (or rub it sideways) to drop food. Birds fly in to eat it.
- Swipe across a glowing feather to collect it. Drag on empty ground to turn the view.
- Use feathers to open new places (작은 물가 · 광장 벤치 · 맑은 강가 · 어두운 숲) and to buy shop items (횃대, 모이통, 새집, 수집기, 청소부).
- Place all five shop items in a place to unlock its hidden object (???).
- Progress is saved in the browser's localStorage.

## 로컬에서 실행

Open `index.html` in a browser, or serve the folder:

```bash
python3 -m http.server 8000
```

Then go to http://localhost:8000.

## 배포 (GitHub Pages)

1. In the GitHub repository, open **Settings → Pages**.
2. Under **Build and deployment → Source**, choose **Deploy from a branch**.
3. Pick the **main** branch and the **/ (root)** folder, then save.
4. After a minute or two, the game is live at `https://dprm1004001.github.io/bird-yard/`.

Pages on a free account requires a **Public** repository.

You can also deploy the folder as-is to any static host, such as Netlify or Vercel.
