# Frontend Loyihalar

Brauzer API'lari va zamonaviy UI muhandisligiga urg'u bergan, portfolioda ajralib turadigan React/TypeScript loyihalari.

## 🟢 Beginner

### 🔹 Markdown Editor with Live Preview
- **🎯 Maqsad:** Foydalanuvchi chap tarafda Markdown yozadi, o'ng tarafda esa real vaqtda HTML natijani ko'radi. Har bir developer'ga kerak bo'ladigan, hujjatlashtirish uchun ishlatiladigan real vosita.
- **📚 Nimani o'rganasiz:** Controlled component'lar, `debounce` bilan performance optimizatsiya, third-party parser integratsiyasi, XSS'dan himoya (sanitize).
- **⚙️ Asosiy funksiyalar:** Split-pane layout; real-time preview; syntax highlighting kod bloklarida; localStorage'ga avtomatik saqlash; `.md` faylni export qilish.
- **🛠️ Tech tavsiya:** React + TypeScript, `marked` yoki `markdown-it`, `DOMPurify`, `highlight.js`.
- **🚀 Qo'shimcha:** Scroll sync (ikkala panel birga siljiydi), rasmni drag&drop bilan yuklash, bir nechta theme, GitHub Flavored Markdown (jadval, checkbox).

### 🔹 Image Color Palette Extractor
- **🎯 Maqsad:** Foydalanuvchi rasm yuklaydi, ilova undan dominant ranglar palitrasini chiqaradi. Dizaynerlar va brending uchun real foydali.
- **📚 Nimani o'rganasiz:** Canvas API, `getImageData` bilan pixel'larni o'qish, color quantization algoritmlari, RGB↔HEX konvertatsiya.
- **⚙️ Asosiy funksiyalar:** Drag&drop rasm yuklash; 5-8 ta dominant rangni ajratish; HEX kodni bosib nusxalash; palitrani PNG/JSON qilib export.
- **🛠️ Tech tavsiya:** React + TypeScript, Canvas API, `color-thief` (yoki o'z algoritming — median cut).
- **🚀 Qo'shimcha:** Accessible kontrast tekshiruvi (WCAG), Tailwind config generatsiya qilish, gradient taklif qilish.

### 🔹 Keyboard Typing Speed Trainer
- **🎯 Maqsad:** Foydalanuvchining yozish tezligini (WPM) va aniqligini o'lchaydigan trenajyor. O'rgatuvchi va o'ynaladigan, o'zbek va inglizcha matnlarni qo'llab-quvvatlaydi.
- **📚 Nimani o'rganasiz:** Keyboard event'larni real vaqtda ushlash, timing hisoblash, state machine, visual feedback rendering.
- **⚙️ Asosiy funksiyalar:** Real vaqtda WPM va accuracy; xato belgilarni rang bilan ajratish; timer va progress bar; natijalar tarixi.
- **🛠️ Tech tavsiya:** React + TypeScript, `Date.now()` timing, CSS transitions.
- **🚀 Qo'shimcha:** O'zbekcha/inglizcha rejimlar, kod yozish rejimi (dasturchilar uchun), heatmap qaysi tugma ko'p xato qilinishini ko'rsatadi.

### 🔹 Pomodoro Timer with Analytics
- **🎯 Maqsad:** Oddiy pomodoro emas — u fokus seanslaringni yozib boradi va statistika chiqaradi. Real produktivlik vositasi.
- **📚 Nimani o'rganasiz:** `setInterval` va aniq timing (drift'siz), Notification API, IndexedDB/localStorage bilan data persistence, chart rendering.
- **⚙️ Asosiy funksiyalar:** Ishlash/tanaffus sikllari; brauzer notification'lari; kunlik/haftalik statistika grafigi; loyiha bo'yicha vaqt hisobi.
- **🛠️ Tech tavsiya:** React + TypeScript, Notification API, `recharts` yoki `chart.js`.
- **🚀 Qo'shimcha:** Streak tracking, PWA qilib telefonga o'rnatish, tab title'da timer, ambient tovushlar.

### 🔹 CSS Animation Playground
- **🎯 Maqsad:** Foydalanuvchi vizual ravishda CSS animatsiya/transition yaratadi va tayyor kodni oladi. Frontend developerlar uchun real vosita.
- **📚 Nimani o'rganasiz:** CSS keyframes, cubic-bezier, controlled slider'lar, kod generatsiyasi, live preview.
- **⚙️ Asosiy funksiyalar:** Vizual timeline editor; bezier curve tahrirlagich; live preview; tayyor CSS kodni nusxalash.
- **🛠️ Tech tavsiya:** React + TypeScript, CSS custom properties, `<canvas>` bezier chizish uchun.
- **🚀 Qo'shimcha:** Preset kutubxona, animatsiyani GIF/video qilib export, share qilish uchun URL state.

### 🔹 Interactive Resume / Portfolio Builder
- **🎯 Maqsad:** Foydalanuvchi forma to'ldiradi, ilova chiroyli, chop etsa bo'ladigan CV generatsiya qiladi. O'zing va boshqalar uchun real ishlatiladigan.
- **📚 Nimani o'rganasiz:** Form state boshqaruvi, live preview, `@media print` bilan print-friendly layout, PDF export.
- **⚙️ Asosiy funksiyalar:** Bo'limlarni qo'shish/o'chirish/reorder; live preview; bir nechta template; PDF export.
- **🛠️ Tech tavsiya:** React + TypeScript, `react-hook-form`, `react-to-print` yoki `html2pdf`.
- **🚀 Qo'shimcha:** URL orqali share, JSON import/export, ATS-friendly rejim, drag&drop bo'limlarni tartiblash.

## 🟡 Intermediate

### 🔹 Kanban Board (Drag & Drop)
- **🎯 Maqsad:** Trello uslubidagi vazifa taxtasi — ustunlar orasida kartochkalarni sudrab ko'chirish. Jamoaviy ish uchun real klassik loyiha.
- **📚 Nimani o'rganasiz:** Drag & Drop mexanikasi, murakkab nested state, optimistic update, persistence.
- **⚙️ Asosiy funksiyalar:** Ustun va kartochka yaratish/tahrirlash; drag&drop bilan ko'chirish; label va priority; localStorage yoki backend'ga saqlash.
- **🛠️ Tech tavsiya:** React + TypeScript, `@dnd-kit` (yoki `react-beautiful-dnd`), Zustand.
- **🚀 Qo'shimcha:** Bir nechta board, filtr/qidiruv, deadline eslatmalari, real-time collaboration (WebSocket).

### 🔹 Spreadsheet Clone (Formula bilan)
- **🎯 Maqsad:** Excel/Google Sheets kabi mini jadval — kataklar orasida formula (`=A1+B2`) hisoblaydi. Formula parser va dependency grafi tufayli juda o'rgatuvchi.
- **📚 Nimani o'rganasiz:** Formula parsing (tokenizer + evaluator), dependency graph, circular reference aniqlash, katta grid'ni samarali render qilish.
- **⚙️ Asosiy funksiyalar:** Grid katakni tahrirlash; formula qo'llab-quvvatlash (`SUM`, `+`, cell reference); avtomatik qayta hisoblash; keyboard navigatsiya.
- **🛠️ Tech tavsiya:** React + TypeScript, o'z formula parser, virtualized rendering (`react-window`).
- **🚀 Qo'shimcha:** CSV import/export, katak formatlash, undo/redo, ko'p sheet.

### 🔹 Music Player with Web Audio Visualizer
- **🎯 Maqsad:** MP3 pleyer — musiqa chalinganda tovushni real vaqtda vizualizatsiya qiladi (frequency bar, waveform). Web Audio API'ning kuchini ko'rsatadi.
- **📚 Nimani o'rganasiz:** Web Audio API, `AnalyserNode` bilan FFT, `requestAnimationFrame` bilan Canvas rendering, audio buffer boshqaruvi.
- **⚙️ Asosiy funksiyalar:** Lokal fayl yuklash va chalish; frequency spectrum vizualizatsiya; playlist; volume/seek boshqaruvi.
- **🛠️ Tech tavsiya:** React + TypeScript, Web Audio API, Canvas API.
- **🚀 Qo'shimcha:** Ekvalayzer (biquad filter), turli vizualizatsiya rejimlari, mikrofon input vizualizatsiyasi, Media Session API.

### 🔹 Infinite Scroll Image Gallery (Masonry + Lazy Load)
- **🎯 Maqsad:** Pinterest uslubidagi galereya — pastga siljiganda avtomatik yangi rasmlar yuklanadi, masonry layout'da joylashadi. Performance muhandisligi mashqi.
- **📚 Nimani o'rganasiz:** IntersectionObserver, lazy loading, masonry layout algoritmi, image preloading, virtualizatsiya.
- **⚙️ Asosiy funksiyalar:** Masonry grid; infinite scroll (IntersectionObserver); blur-up placeholder; qidiruv/filtr.
- **🛠️ Tech tavsiya:** React + TypeScript, IntersectionObserver API, Unsplash/Pexels API.
- **🚀 Qo'shimcha:** Lightbox modal, DOM virtualizatsiya (minglab rasm uchun), favorites, download.

### 🔹 Collaborative Drawing Canvas
- **🎯 Maqsad:** Bir nechta odam bir vaqtda bitta canvas'da chizadigan doska. WebRTC/WebSocket bilan real-time sinxronizatsiya.
- **📚 Nimani o'rganasiz:** Canvas drawing API, WebSocket yoki WebRTC data channel, real-time state sync, pointer event'lar.
- **⚙️ Asosiy funksiyalar:** Bemalol chizish (brush/rang/qalinlik); real vaqtda boshqalarni ko'rish; undo/redo; canvas'ni PNG export.
- **🛠️ Tech tavsiya:** React + TypeScript, Canvas API, `socket.io` yoki WebRTC (PeerJS).
- **🚀 Qo'shimcha:** Kursorlarni ko'rsatish (presence), shakl vositalari, cheksiz zoomable canvas, room'lar.

### 🔹 Custom Video Player
- **🎯 Maqsad:** Noldan yozilgan, to'liq stillashtirilgan video pleyer — o'z controllari, subtitr, tezlik, PiP bilan. Native `<video>`'ni chuqur o'rganish.
- **📚 Nimani o'rganasiz:** HTMLMediaElement API, custom controls, keyboard shortcut'lar, Picture-in-Picture API, fullscreen API.
- **⚙️ Asosiy funksiyalar:** Play/pause/seek/volume; progress bar buffer ko'rsatkichi bilan; tezlikni o'zgartirish; subtitr (WebVTT).
- **🛠️ Tech tavsiya:** React + TypeScript, HTMLMediaElement API, Fullscreen/PiP API.
- **🚀 Qo'shimcha:** Chapter markerlar, thumbnail preview seek'da, HLS streaming (`hls.js`), keyboard shortcut overlay.

### 🔹 Interactive Data Table (Sort/Filter/Group)
- **🎯 Maqsad:** Kuchli jadval komponenti — saralash, filtrlash, guruhlash, ustunlarni yashirish/kengaytirish bilan. Har qanday admin panelda kerak.
- **📚 Nimani o'rganasiz:** Generic TypeScript komponent dizayni, murakkab state, memoization, column virtualization.
- **⚙️ Asosiy funksiyalar:** Multi-column sort; per-column filter; pagination; inline tahrirlash.
- **🛠️ Tech tavsiya:** React + TypeScript, `@tanstack/table`, `@tanstack/virtual`.
- **🚀 Qo'shimcha:** CSV export, ustun reorder (drag), saqlangan view'lar, server-side pagination.

## 🔴 Advanced

### 🔹 Code Editor with Syntax Highlighting (Monaco)
- **🎯 Maqsad:** Brauzerda ishlaydigan mini IDE — syntax highlighting, autocomplete, bir nechta fayl, hatto brauzerda kod bajarish. VS Code'ning yuragi Monaco bilan.
- **📚 Nimani o'rganasiz:** Monaco Editor integratsiyasi, LSP tushunchalari, Web Worker'da kod bajarish, virtual file system.
- **⚙️ Asosiy funksiyalar:** Monaco editor bir nechta til bilan; fayl daraxti; JS/TS'ni brauzerda ishlatish; natija konsoli.
- **🛠️ Tech tavsiya:** React + TypeScript, `monaco-editor`, Web Workers, `esbuild-wasm`.
- **🚀 Qo'shimcha:** NPM paketlarini import qilish (`esm.sh`), share qilinadigan link, live preview (StackBlitz uslubi), theme.

### 🔹 Real-Time Collaborative Text Editor (CRDT)
- **🎯 Maqsad:** Google Docs kabi — bir nechta foydalanuvchi bitta hujjatni bir vaqtda tahrirlaydi, konfliktlarsiz birlashadi. CRDT — distributed systems'ning eng qiziq mavzusi.
- **📚 Nimani o'rganasiz:** CRDT algoritmlari (yoki OT), conflict-free merge, presence/cursor sync, WebSocket sinxronizatsiya.
- **⚙️ Asosiy funksiyalar:** Bir nechta kursorli real-time tahrirlash; offline redaktura keyin merge; foydalanuvchi presence; hujjat tarixi.
- **🛠️ Tech tavsiya:** React + TypeScript, `Yjs` (CRDT), `y-websocket`, ProseMirror/TipTap.
- **🚀 Qo'shimcha:** Comment thread'lar, suggest rejimi, versiya tarixi va diff, rich text formatting.

### 🔹 Data Dashboard with Charts + Virtualized Table (10k+ rows)
- **🎯 Maqsad:** Analitik dashboard — interaktiv grafiklar va 10 mingdan ortiq qatorli, tez ishlaydigan jadval. Real BI vositasi darajasidagi performance.
- **📚 Nimani o'rganasiz:** Row/column virtualizatsiya, katta datasetni samarali qayta ishlash, chart kutubxonalari, memoization strategiyalari.
- **⚙️ Asosiy funksiyalar:** Bir nechta chart turi (line/bar/pie); 10k+ qatorli virtualized jadval; sana/kategoriya filtrlari; cross-filtering (chartni bosganda jadval filtrlanadi).
- **🛠️ Tech tavsiya:** React + TypeScript, `@tanstack/virtual`, `visx`/`recharts`, Web Worker og'ir hisob uchun.
- **🚀 Qo'shimcha:** Real-time streaming data (WebSocket), dashboard layout'ni drag bilan sozlash, PDF/PNG export, WebGL chart (`deck.gl`).

### 🔹 Diagram / Flowchart Builder
- **🎯 Maqsad:** draw.io kabi node va bog'lanishlarni sudrab yaratadigan diagramma vositasi. Grafik muharrir muhandisligi.
- **📚 Nimani o'rganasiz:** SVG/Canvas rendering, graph data model, snap-to-grid, pan/zoom transform matematikasi, auto-layout.
- **⚙️ Asosiy funksiyalar:** Node yaratish/ko'chirish; node'lar orasida edge chizish; pan/zoom; JSON export/import.
- **🛠️ Tech tavsiya:** React + TypeScript, `React Flow` (yoki noldan SVG), `dagre` auto-layout.
- **🚀 Qo'shimcha:** Turli node turlari, undo/redo (command pattern), Mermaid'ga export, real-time collab.

### 🔹 Mini Figma-like Design Tool
- **🎯 Maqsad:** Vektor dizayn muharriri — shakl chizish, tanlash, transform, layer'lar bilan. Frontend'ning eng murakkab loyihalaridan biri.
- **📚 Nimani o'rganasiz:** Canvas/SVG rendering pipeline, transform matritsalari (translate/scale/rotate), hit-testing, layer va selection tizimi, komandalar tarixi.
- **⚙️ Asosiy funksiyalar:** Shakllar (rect/ellipse/text) chizish; tanlash va resize/rotate handle'lar; layer paneli; pan/zoom.
- **🛠️ Tech tavsiya:** React + TypeScript, Canvas API (yoki `Konva`/`fabric.js`), custom scene graph.
- **🚀 Qo'shimcha:** Multiplayer editing (CRDT), komponent/reuse, export (PNG/SVG), plugin API.

### 🔹 Virtual DOM'ning O'z Implementatsiyasi
- **🎯 Maqsad:** Kichik React klonini noldan yozish — virtual DOM, diffing algoritmi, reconciliation, hatto hooks. Frameworklar ichini tushunishning eng yaxshi yo'li.
- **📚 Nimani o'rganasiz:** Virtual DOM daraxti, diff/reconciliation algoritmi, JSX (`h()` funksiyasi), state va `useState`/`useEffect` mexanikasi.
- **⚙️ Asosiy funksiyalar:** `createElement` va render; VDOM diff va minimal DOM update; funksional komponentlar; oddiy hook'lar.
- **🛠️ Tech tavsiya:** TypeScript (framework'siz), Vite, JSX transform sozlamasi.
- **🚀 Qo'shimcha:** Keyed reconciliation, fragment, event delegatsiya, concurrent rendering g'oyasi, test suite bilan React'ga taqqoslash.

### 🔹 In-Browser Image Editor (Filters + Layers)
- **🎯 Maqsad:** Photoshop-lite — filtr, crop, layer, brush bilan rasm tahrirlash. Web Worker va WebGL bilan tez ishlaydigan.
- **📚 Nimani o'rganasiz:** Canvas pixel manipulyatsiya, convolution filtrlar (blur/sharpen), Web Worker'da og'ir ishlov, ixtiyoriy WebGL/WebGPU shader'lar.
- **⚙️ Asosiy funksiyalar:** Rasm yuklash va crop; filtrlar (brightness/contrast/blur); bir nechta layer; export.
- **🛠️ Tech tavsiya:** React + TypeScript, Canvas API, Web Workers, ixtiyoriy WebGL (`gl-react`).
- **🚀 Qo'shimcha:** Non-destructive tahrirlash (adjustment layer), histogram, WebGPU compute shader, undo/redo.

← [Loyihalar bo'limiga qaytish](./README.md)
