# React Asoslari

> Yechimlar: [solutions/frontend/05-react-fundamentals.md](../solutions/frontend/05-react-fundamentals.md)

Bu fayl React'ning eng asosiy mental modelini qamrab oladi: nega React kerak, JSX qanday ishlaydi, component, props va state, virtual DOM va reconciliation, hamda intervyularda eng ko'p so'raladigan tushunchalar. Tushuntirishlar o'zbekcha, kod va atamalar inglizcha (TypeScript misollarda).

## Mundarija

- [React nima va nega kerak](#react-nima-va-nega-kerak)
- [JSX](#jsx)
- [Function va Class component](#function-va-class-component)
- [Props](#props)
- [State va Props farqi](#state-va-props-farqi)
- [Virtual DOM, reconciliation va diffing](#virtual-dom-reconciliation-va-diffing)
- [List'da key](#listda-key)
- [Conditional rendering](#conditional-rendering)
- [List rendering](#list-rendering)
- [Controlled vs uncontrolled, form](#controlled-vs-uncontrolled-form)
- [React event (synthetic event)](#react-event-synthetic-event)
- [One-way data flow va lifting state up](#one-way-data-flow-va-lifting-state-up)
- [Composition vs inheritance](#composition-vs-inheritance)
- [Fragment](#fragment)
- [Portal](#portal)
- [Ref (asosiy)](#ref-asosiy)
- [Render qoidalari (pure)](#render-qoidalari-pure)
- [Nima re-render keltiradi](#nima-re-render-keltiradi)
- [Lifecycle (hook tilida)](#lifecycle-hook-tilida)
- [StrictMode](#strictmode)
- [Masalalar](#masalalar)

---

## React nima va nega kerak

**💡 Tushuncha:** React — bu UI qurish uchun mo'ljallangan JavaScript library. Uning ikki asosiy g'oyasi bor: **declarative** (siz UI qanday ko'rinishi kerakligini aytasiz, DOM'ni qo'lda o'zgartirmaysiz) va **component-based** (UI mustaqil, qayta ishlatiladigan bo'laklarga bo'linadi).

Imperative yondashuvda siz DOM'ni qadama-qadam o'zgartirasiz: `element.textContent = "..."`, `appendChild` va hokazo. Declarative yondashuvda siz faqat "state shu bo'lsa, UI shunday bo'lsin" deb yozasiz, React esa DOM'ni o'zi yangilaydi.

### ❓ React library'mi yoki framework?

**✅ Javob:** React rasman **library** — u faqat UI (view) qatlamiga javob beradi. Routing, data fetching, state management kabilarni o'zingiz tanlaysiz (React Router, TanStack Query, Redux/Zustand). Next.js kabi narsalar React ustiga qurilgan **framework**lardir. Bu farq intervyuda yaxshi javob beradi: React "view library", to'liq framework emas.

### ❓ "Declarative" deganda nimani nazarda tutasiz?

**✅ Javob:** Declarative kodda siz natijani tasvirlaysiz, jarayonni emas. Masalan, `isLoggedIn ? <Dashboard /> : <Login />` — siz "agar login bo'lsa Dashboard ko'rinsin" deysiz, lekin eski elementni o'chirib, yangisini qo'shish kabi DOM amallarini yozmaysiz. React state o'zgarganda kerakli minimal DOM yangilanishini o'zi hisoblaydi.

**⚠️ Ehtiyot bo'l:** React DOM'ni o'zi boshqaradi degani — siz `document.getElementById(...).innerHTML = ...` bilan React boshqaradigan DOM'ga aralashmasligingiz kerak. Bu reconciliation'ni buzadi.

---

## JSX

**💡 Tushuncha:** JSX — JavaScript ichida HTML'ga o'xshash sintaksis. Bu brauzer tushunadigan narsa emas; build vaqtida (Babel/SWC/TypeScript) u `React.createElement(...)` chaqiruvlariga **compile** bo'ladi.

```tsx
const el = <h1 className="title">Salom</h1>;

// yuqoridagi quyidagiga compile bo'ladi:
const el2 = React.createElement("h1", { className: "title" }, "Salom");
```

`React.createElement` esa oddiy JS object qaytaradi — bu **React element**, "nima render bo'lishi kerakligi"ning yengil tasviri (description). Bu object virtual DOM tugunidir.

### ❓ Nega JSX'da `class` emas, `className` yoziladi?

**✅ Javob:** JSX oxir-oqibat JavaScript object'iga aylanadi va `class` JavaScript'da reserved keyword. Shuning uchun React DOM property nomlaridan foydalanadi: `className`, `htmlFor` (`for` o'rniga). Atributlar camelCase'da yoziladi: `onClick`, `tabIndex`.

### ❓ JSX ichida `{}` nima qiladi?

**✅ Javob:** Jingalak qavslar JSX ichiga JavaScript **expression** kiritadi — qiymat qaytaradigan har qanday ifoda: `{user.name}`, `{2 + 2}`, `{items.map(...)}`, `{isOpen && <Menu />}`. E'tibor bering: bu **expression**, **statement** emas. `if`, `for`, `switch` to'g'ridan-to'g'ri yozilmaydi — buning o'rniga ternary yoki `&&` ishlatiladi.

### ❓ Nega JSX bitta root element qaytarishi kerak?

**✅ Javob:** Chunki `return` bitta qiymat qaytaradi va JSX bitta `createElement` chaqiruvi daraxtiga aylanadi. Bir nechta sibling element'ni qaytarish uchun ularni bitta parent'ga (yoki `Fragment`'ga, ya'ni `<>...</>`'ga) o'rab qo'yish kerak.

**⚠️ Ehtiyot bo'l:** JSX'da `{0 && <X/>}` yozsangiz, ekranda `0` chiqib qoladi, chunki `0` falsy lekin renderlanadigan qiymat. Buning o'rniga `{count > 0 && <X/>}` yoki `{!!count && <X/>}` yozing.

---

## Function va Class component

**💡 Tushuncha:** Component — props qabul qilib, React element qaytaradigan funksiya (yoki class). Zamonaviy React'da **function component** standart hisoblanadi; class component eski kod bazalarda uchraydi.

```tsx
type GreetingProps = { name: string };

// Function component (zamonaviy, tavsiya etiladi)
function Greeting({ name }: GreetingProps) {
  return <h1>Salom, {name}</h1>;
}

// Class component (eski uslub)
class GreetingClass extends React.Component<GreetingProps> {
  render() {
    return <h1>Salom, {this.props.name}</h1>;
  }
}
```

### ❓ Nega function component class'dan afzal?

**✅ Javob:** Function component'lar soddaroq (boilerplate kam, `this` muammosi yo'q), hooks orqali state va lifecycle'ni to'liq qo'llab-quvvatlaydi, logikani qayta ishlatish osonroq (custom hooks orqali), va React jamoasi kelajakdagi optimizatsiyalarni (masalan, React Compiler) function component'larga qaratgan. Hooks paydo bo'lgandan beri class'ning amaliy afzalligi qolmadi.

### ❓ Component nomi nega katta harf bilan boshlanishi kerak?

**✅ Javob:** JSX'da kichik harfli teg (`<div>`) DOM element deb, katta harfli teg (`<Greeting>`) component deb talqin qilinadi. Agar component'ni `greeting` deb nomlasangiz, React uni noma'lum HTML teg deb o'ylaydi va render bo'lmaydi.

---

## Props

**💡 Tushuncha:** Props (properties) — parent component'dan child'ga uzatiladigan ma'lumot. Props **read-only**: child uni o'zgartira olmaydi. Bu React'ning one-way data flow tamoyilining asosi.

```tsx
type ButtonProps = {
  label: string;
  onClick: () => void;
  children?: React.ReactNode;
};

function Button({ label, onClick, children }: ButtonProps) {
  return <button onClick={onClick}>{label} {children}</button>;
}
```

### ❓ `children` prop nima?

**✅ Javob:** `children` — component teglari orasidagi narsa. `<Card>matn</Card>` yozilganda `matn` `props.children`'ga tushadi. Bu composition'ning asosiy mexanizmi: component o'z ichidagi kontentni bilmasdan uni render qila oladi.

```tsx
function Card({ children }: { children: React.ReactNode }) {
  return <div className="card">{children}</div>;
}
// <Card><h2>Sarlavha</h2></Card>
```

### ❓ Prop drilling nima va u nima uchun muammo?

**✅ Javob:** Prop drilling — bir prop'ni bir necha qatlam component orqali, oraliq component'lar unga muhtoj bo'lmasa ham, faqat pastga uzatish uchun o'tkazib yuborish. Bu kodni qiyinlashtiradi va refactor'ni murakkablashtiradi. Yechimi: **Context API** (global-ga yaqin ma'lumot uchun), composition (`children` orqali), yoki state management library.

**⚠️ Ehtiyot bo'l:** Props'ni hech qachon to'g'ridan-to'g'ri o'zgartirmang (`props.x = 5`). Ular read-only va mutate qilish bug keltiradi. O'zgaruvchan qiymat kerak bo'lsa, state ishlating.

---

## State va Props farqi

**💡 Tushuncha:** **Props** — tashqaridan (parent'dan) keladi va o'zgarmas (read-only). **State** — component'ning o'z ichidagi, vaqt o'tishi bilan o'zgaradigan ma'lumoti, va uni o'zgartirish re-render keltiradi.

```tsx
function Counter({ step }: { step: number }) {  // step — prop
  const [count, setCount] = React.useState(0);   // count — state
  return <button onClick={() => setCount(count + step)}>{count}</button>;
}
```

### ❓ State va props farqini bir gapda ayting.

**✅ Javob:** Props — parent boshqaradigan, o'zgarmas input; state — component o'zi boshqaradigan, o'zgaruvchan ichki ma'lumot. Ikkalasi ham o'zgarganda re-render bo'ladi, lekin props'ni faqat parent o'zgartira oladi, state'ni esa component o'zi.

### ❓ State'ni to'g'ridan-to'g'ri o'zgartirsa bo'ladimi? `count++` qilsa?

**✅ Javob:** Yo'q. State'ni faqat setter (`setCount`) orqali yangilash kerak. To'g'ridan-to'g'ri mutate qilish (`count++` yoki `arr.push(...)`) re-render keltirmaydi va React'ning ichki holatini buzadi. Har doim yangi qiymat/yangi object bering.

---

## Virtual DOM, reconciliation va diffing

**💡 Tushuncha:** Virtual DOM (VDOM) — real DOM'ning yengil, JavaScript object ko'rinishidagi nusxasi. State o'zgarganda React yangi VDOM daraxti yaratadi, uni eskisi bilan solishtiradi (**diffing**), va faqat o'zgargan qismlarni real DOM'ga qo'llaydi. Bu jarayonning butuni **reconciliation** deyiladi.

### ❓ Nega VDOM tezroq? Real DOM bilan nima muammo?

**✅ Javob:** Real DOM'ni manipulyatsiya qilish qimmat (layout, reflow, repaint). Agar har bir o'zgarishda butun DOM'ni qayta qursak, juda sekin bo'ladi. VDOM — bu xotiradagi arzon object'lar; React ularni solishtirib, real DOM'ga **minimal** yangilanish kiritadi. Ya'ni VDOM sehrli tarzda tez emas, balki u **keraksiz real DOM amallarini kamaytiradi**.

### ❓ Diffing algoritmi qanday ishlaydi (heuristika)?

**✅ Javob:** To'liq daraxt solishtirish O(n³) bo'lardi, shuning uchun React O(n) heuristikalardan foydalanadi:
1. **Element type o'zgarsa** (masalan `<div>` → `<span>`), eski subtree butunlay tashlanadi va yangisi quriladi.
2. **Type bir xil bo'lsa**, faqat o'zgargan atribut/property'lar yangilanadi.
3. **List'lar uchun `key`** ishlatiladi — qaysi element qaysisiga mos kelishini aniqlash uchun.

### ❓ Reconciliation va re-render bir narsami?

**✅ Javob:** Yo'q. **Re-render** — component funksiyasining qayta chaqirilishi va yangi VDOM hosil bo'lishi. **Reconciliation** — yangi va eski VDOM'ni solishtirib, real DOM'ga qaysi o'zgarishlarni qo'llashni aniqlash. Re-render ko'p bo'lishi mumkin, lekin reconciliation tufayli real DOM kam o'zgaradi.

**⚠️ Ehtiyot bo'l:** Re-render har doim DOM o'zgarishini anglatmaydi. Agar render natijasi avvalgidek bo'lsa, React real DOM'ga tegmaydi. Shuning uchun "re-render = sekin" degan tushuncha noto'g'ri.

---

## List'da key

**💡 Tushuncha:** `key` — list'dagi har bir element uchun barqaror, noyob identifikator. React undan diffing paytida qaysi element qo'shilgani, o'chirilgani yoki ko'chirilganini aniqlash uchun foydalanadi.

```tsx
function TodoList({ todos }: { todos: { id: string; text: string }[] }) {
  return (
    <ul>
      {todos.map((t) => (
        <li key={t.id}>{t.text}</li>
      ))}
    </ul>
  );
}
```

### ❓ Nega array index'ni key qilib ishlatish yomon?

**✅ Javob:** Index — element'ning emas, uning hozirgi pozitsiyasining identifikatori. Agar list o'rtasiga element qo'shilsa yoki o'chirilsa yoki tartibi o'zgarsa, index'lar siljiydi va React noto'g'ri element'larni "bir xil" deb hisoblaydi. Bu xato UI'ga olib keladi: input qiymatlari aralashib ketadi, animatsiyalar buziladi, noto'g'ri DOM qayta ishlatiladi. Barqaror, ma'lumotga bog'liq `id` ishlating.

### ❓ Index'ni key qilib ishlatish qachon to'g'ri?

**✅ Javob:** Agar list **statik** bo'lsa (hech qachon o'zgarmaydi, qayta tartiblanmaydi, qo'shilmaydi/o'chirilmaydi) va element'larda o'z ID'si bo'lmasa, index maqbul. Lekin shubha bo'lsa, doim barqaror ID ishlating.

**⚠️ Ehtiyot bo'l:** `key` — bu prop emas; child ichida `props.key` orqali o'qib bo'lmaydi. U faqat React uchun. Agar qiymat kerak bo'lsa, alohida prop sifatida ham uzating.

---

## Conditional rendering

**💡 Tushuncha:** Shartga qarab turli UI ko'rsatish. JSX expression bo'lgani uchun ternary, `&&`, yoki funksiyaning boshida `if` ishlatiladi.

```tsx
function Status({ loading, error, data }: { loading: boolean; error?: string; data?: string }) {
  if (loading) return <Spinner />;
  if (error) return <p className="err">{error}</p>;
  return <p>{data}</p>;
}

// JSX ichida:
// {isOpen && <Modal />}
// {count > 0 ? <List /> : <Empty />}
```

### ❓ `&&` bilan conditional render'da qanday tuzoq bor?

**✅ Javob:** Chap tomon `0` yoki `NaN` kabi renderlanadigan falsy qiymat bo'lsa, u ekranga chiqadi. `{items.length && <List/>}` — `items.length === 0` bo'lsa ekranda `0` ko'rinadi. Yechimi: `{items.length > 0 && <List/>}`.

---

## List rendering

**💡 Tushuncha:** Massivni UI'ga aylantirish uchun `.map()` ishlatiladi, har bir element'ga `key` beriladi.

```tsx
const users = [{ id: 1, name: "Ali" }, { id: 2, name: "Vali" }];

<ul>
  {users.map((u) => <li key={u.id}>{u.name}</li>)}
</ul>
```

### ❓ Nega `.map()` ishlatamiz, `.forEach()` emas?

**✅ Javob:** JSX ichiga renderlash uchun **qiymat qaytaradigan** narsa kerak. `.map()` yangi massiv (element'lar massivi) qaytaradi, JSX uni render qiladi. `.forEach()` esa `undefined` qaytaradi — render qilish uchun hech narsa yo'q.

---

## Controlled vs uncontrolled, form

**💡 Tushuncha:** **Controlled** input'ning qiymati React state'da turadi (`value` + `onChange`). **Uncontrolled** input'ning qiymatini DOM'ning o'zi boshqaradi, React unga `ref` orqali kerak bo'lganda murojaat qiladi.

```tsx
// Controlled
function ControlledInput() {
  const [name, setName] = React.useState("");
  return <input value={name} onChange={(e) => setName(e.target.value)} />;
}

// Uncontrolled
function UncontrolledInput() {
  const ref = React.useRef<HTMLInputElement>(null);
  const submit = () => alert(ref.current?.value);
  return (
    <>
      <input ref={ref} defaultValue="" />
      <button onClick={submit}>Yubor</button>
    </>
  );
}
```

### ❓ Controlled va uncontrolled qaysi biri afzal?

**✅ Javob:** Controlled odatda afzal: qiymat har doim state'da, shuning uchun validatsiya, formatlash, shartli o'chirib qo'yish (disable) oson. Uncontrolled qiymat kam kerak bo'lganda (masalan oddiy submit) yoki katta form'da har bosishda re-render bo'lishini xohlamasangiz qulay. File input (`<input type="file">`) doim uncontrolled.

**⚠️ Ehtiyot bo'l:** Bir input'da `value` (controlled) va `defaultValue` (uncontrolled) ni birga ishlatmang. Shuningdek, `value` berib `onChange` bermasangiz, React ogohlantirish beradi va input "qotib qoladi" (read-only bo'lib qoladi).

---

## React event (synthetic event)

**💡 Tushuncha:** React DOM event'larni `SyntheticEvent` deb nomlangan, brauzerlararo bir xil ishlaydigan wrapper'ga o'raydi. Bu native event ustidagi cross-browser qatlam.

```tsx
function Form() {
  const handle = (e: React.FormEvent) => {
    e.preventDefault(); // sahifa qayta yuklanmaydi
  };
  return <form onSubmit={handle} />;
}
```

### ❓ Synthetic event nima va nega kerak?

**✅ Javob:** SyntheticEvent — native event ustidagi yagona, brauzerlardan mustaqil API. U bir xil interfeys (`e.target`, `e.preventDefault()`, `e.stopPropagation()`) beradi va React event delegation'dan foydalanadi: har element'ga alohida listener qo'yish o'rniga, root'da bitta listener orqali event'larni boshqaradi (perf afzallik).

### ❓ React 17+'da event delegation qayerga biriktiriladi?

**✅ Javob:** React 17'gacha event'lar `document`'ga biriktirilardi. React 17+'da esa React render qiladigan **root container**'ga biriktiriladi. Bu bir sahifada bir nechta React versiyasi/ilovasi bo'lsa konfliktni kamaytiradi.

---

## One-way data flow va lifting state up

**💡 Tushuncha:** React'da ma'lumot **bir yo'nalishda** — parent'dan child'ga props orqali oqadi. Child parent'ga ta'sir qilish uchun callback (event handler prop) chaqiradi. Agar ikki child bitta state'ni baham ko'rishi kerak bo'lsa, o'sha state ularning eng yaqin umumiy parent'iga ko'tariladi — bu **lifting state up**.

```tsx
function Parent() {
  const [value, setValue] = React.useState("");
  return (
    <>
      <Input value={value} onChange={setValue} />
      <Preview value={value} />
    </>
  );
}
```

### ❓ Lifting state up'ni qachon qilasiz?

**✅ Javob:** Ikki yoki undan ko'p component bir xil o'zgaruvchan ma'lumotni baham ko'rishi kerak bo'lganda. State'ni ularning eng yaqin umumiy parent'iga ko'tarasiz, parent uni props orqali pastga uzatadi va o'zgartirish uchun callback beradi. Bu single source of truth'ni saqlaydi.

---

## Composition vs inheritance

**💡 Tushuncha:** React inheritance o'rniga **composition**'ni tavsiya qiladi. Component'larni bir-biriga `props` va `children` orqali joylashtirib (compose qilib) qayta ishlatasiz, class inheritance ishlatmaysiz.

```tsx
function Dialog({ title, children }: { title: string; children: React.ReactNode }) {
  return (
    <div className="dialog">
      <h2>{title}</h2>
      <div>{children}</div>
    </div>
  );
}
// <Dialog title="Ogohlantirish"><p>Davom etasizmi?</p></Dialog>
```

### ❓ Nega React'da inheritance ishlatilmaydi?

**✅ Javob:** UI muammolarining deyarli barchasi composition bilan toza hal bo'ladi: umumiy "qobiq" component yaratib, ichini `children` yoki props orqali to'ldirasiz. Inheritance qattiq bog'lanish (tight coupling) keltiradi va React jamoasi hech qachon component'lar uchun inheritance'ni tavsiya qilmagan. "Has-a" (composition) "is-a" (inheritance)'dan moslashuvchan.

---

## Fragment

**💡 Tushuncha:** `Fragment` — DOM'ga qo'shimcha element qo'shmasdan bir nechta child'ni guruhlash usuli. Qisqa yozuvi: `<>...</>`.

```tsx
function Row() {
  return (
    <>
      <td>Ism</td>
      <td>Familiya</td>
    </>
  );
}
```

### ❓ Nega Fragment kerak, `<div>` bilan o'rab qo'ymaymizmi?

**✅ Javob:** Component bitta root qaytarishi kerak, lekin har doim `<div>` qo'shsak, keraksiz DOM tugunlari paydo bo'ladi — bu layout'ni (flexbox/grid, `<table>` strukturasi) buzishi mumkin. Fragment guruhlaydi, lekin DOM'da hech narsa qoldirmaydi. Agar `key` kerak bo'lsa, to'liq shaklini ishlating: `<React.Fragment key={id}>`.

---

## Portal

**💡 Tushuncha:** Portal — child'ni parent DOM iyerarxiyasidan **tashqaridagi** boshqa DOM tuguniga render qilish imkonini beradi, lekin React daraxtida u o'z joyida qoladi (event'lar, context normal ishlaydi).

```tsx
import { createPortal } from "react-dom";

function Modal({ children }: { children: React.ReactNode }) {
  return createPortal(
    <div className="modal-overlay">{children}</div>,
    document.body
  );
}
```

### ❓ Portal qachon kerak?

**✅ Javob:** Modal, tooltip, dropdown, toast kabi UI'lar uchun. Bunday element'lar `overflow: hidden` yoki `z-index` muammolaridan qochish uchun DOM'ning yuqori qismiga (odatda `document.body`'ga) render bo'lishi kerak, lekin React mantiqida ular o'z component'ida turaverishi qulay. Muhim: event'lar React daraxti bo'yicha (DOM bo'yicha emas) ko'tariladi.

---

## Ref (asosiy)

**💡 Tushuncha:** `ref` — DOM element'ga yoki mutable qiymatga to'g'ridan-to'g'ri murojaat qilish usuli. `ref.current` o'zgarishi re-render keltirmaydi.

```tsx
function TextInput() {
  const inputRef = React.useRef<HTMLInputElement>(null);
  React.useEffect(() => {
    inputRef.current?.focus(); // mount'da fokuslash
  }, []);
  return <input ref={inputRef} />;
}
```

### ❓ Ref qachon ishlatiladi?

**✅ Javob:** Asosan declarative React modelidan "chiqish" kerak bo'lganda: DOM element'ni fokuslash, scroll qilish, o'lchovini olish, media (`video`/`canvas`) bilan ishlash, yoki re-render keltirmaydigan mutable qiymat saqlash (masalan, timer ID, oldingi qiymat). State o'rniga ref'ni faqat UI'ga ta'sir qilmaydigan ma'lumot uchun ishlating.

**⚠️ Ehtiyot bo'l:** Ref render paytida o'qilmaydi/yozilmaydi (faqat event handler yoki effect ichida). Va `ref.current` o'zgartirilsa UI yangilanmaydi — UI'ga ta'sir qiluvchi ma'lumot uchun state ishlating.

---

## Render qoidalari (pure)

**💡 Tushuncha:** React component'lar render paytida **pure** bo'lishi kerak: bir xil props/state uchun bir xil JSX qaytarishi, va render davomida tashqi narsalarni o'zgartirmasligi (side effect qilmasligi) kerak.

### ❓ "Render pure bo'lishi kerak" nimani anglatadi?

**✅ Javob:** Render funksiyasi faqat o'z input'lari (props, state, context) asosida JSX hisoblashi kerak. Render davomida quyidagilarni qilmaslik kerak: tashqi o'zgaruvchini o'zgartirish, DOM'ga yozish, network so'rov yuborish, `Math.random()`/`Date.now()` natijasini render strukturasiga to'g'ridan-to'g'ri bog'lab tashlash. Bunday side effect'lar event handler yoki `useEffect` ichida bo'lishi kerak. Pure render React'ga xavfsiz optimizatsiya va qayta-render qilish imkonini beradi.

**⚠️ Ehtiyot bo'l:** Render ichida shartsiz `setState` chaqirish cheksiz loop keltiradi. State'ni event/effect ichida yangilang.

---

## Nima re-render keltiradi

**💡 Tushuncha:** Component qayta render bo'lishining asosiy sabablari: (1) uning state'i o'zgardi, (2) uning parent'i re-render bo'ldi, (3) u tinglagan context qiymati o'zgardi, (4) props o'zgardi (parent re-render natijasida).

### ❓ Parent re-render bo'lsa, barcha child'lar ham re-render bo'ladimi?

**✅ Javob:** Ha — odatiy holatda parent re-render bo'lsa, uning barcha child'lari (props o'zgarmagan bo'lsa ham) qayta render bo'ladi. Buni `React.memo` bilan to'xtatish mumkin: agar props o'zgarmasa, memoized child re-render bo'lmaydi. Lekin re-render har doim ham qimmat emas — avval o'lchang, keyin optimizatsiya qiling.

### ❓ State'ni bir xil qiymatga set qilsa re-render bo'ladimi?

**✅ Javob:** React `Object.is` bilan solishtiradi. Agar yangi qiymat eskisi bilan bir xil bo'lsa (primitive uchun), React render'ni o'tkazib yuborishi mumkin. Lekin yangi object/array bersangiz (reference boshqa), mazmunan bir xil "ko'rinsa" ham re-render bo'ladi.

---

## Lifecycle (hook tilida)

**💡 Tushuncha:** Function component'da lifecycle alohida metodlar emas, `useEffect` orqali ifodalanadi. Class'dagi `componentDidMount`, `componentDidUpdate`, `componentWillUnmount` o'rniga effect va uning cleanup'i ishlatiladi.

```tsx
React.useEffect(() => {
  // mount + har dependency o'zgarganda (componentDidMount/DidUpdate)
  const id = setInterval(tick, 1000);
  return () => clearInterval(id); // unmount yoki keyingi effect oldidan (componentWillUnmount)
}, [/* deps */]);
```

### ❓ Class lifecycle metodlari hook'da qanday ifodalanadi?

**✅ Javob:**
- `componentDidMount` → `useEffect(() => {...}, [])` (bo'sh dependency array, faqat mount'da).
- `componentDidUpdate` → `useEffect(() => {...}, [deps])` (deps o'zgarganda).
- `componentWillUnmount` → effect ichidagi `return () => {...}` (cleanup).

Hook modelida "lifecycle metodlari" haqida emas, balki "bu effect qaysi qiymatlarga sinxronlanadi" deb o'ylash to'g'riroq.

---

## StrictMode

**💡 Tushuncha:** `<React.StrictMode>` — faqat **development**'da ishlaydigan, potensial muammolarni ochib beruvchi yordamchi. U production'da hech narsa qilmaydi va UI chiqarmaydi.

### ❓ StrictMode nima qiladi va nega effect ikki marta ishlaydi?

**✅ Javob:** Development'da StrictMode component'larni (va effect'larni) qasddan **ikki marta** mount qiladi — bu pure bo'lmagan render va cleanup'siz effect kabi bug'larni ochib beradi. Ya'ni `useEffect` mount → unmount → mount ketma-ketligini ko'rsatadi. Bu real bug emas; agar effect'ingizning cleanup'i to'g'ri bo'lsa, ikki marta ishlash muammo tug'dirmaydi. Production'da bu takror yo'q.

**⚠️ Ehtiyot bo'l:** StrictMode'dagi ikki marta chaqiruv "React'da bug" emas — bu sizning effect'ingiz cleanup'siz yoki idempotent emasligini ko'rsatadigan signal. Cleanup yozsangiz, muammo yo'qoladi.

---

## Masalalar

> Yechimlar: [solutions/frontend/05-react-fundamentals.md](../solutions/frontend/05-react-fundamentals.md)

1. **Counter va step:** `step` prop'ini qabul qiluvchi `Counter` component yozing. Tugma har bosilganda hisob `step`'ga oshsin. State'ni functional update bilan yangilang.

2. **Controlled form:** `name` va `email` maydonli controlled form yozing. Ikkala input ham state'ga bog'liq bo'lsin, submit'da `e.preventDefault()` chaqirib, qiymatlarni `console.log` qiling.

3. **Key bug'ni tuzating:** Berilgan list `key={index}` ishlatadi va element o'chirilganda input qiymatlari aralashib ketadi. Muammoni `key` orqali to'g'ri ID bilan tuzating va nega index yomon ekanini bir-ikki gapda izohlang.

4. **Lifting state up:** Ikkita `TemperatureInput` (biri Celsius, biri Fahrenheit) bir xil haroratni ko'rsatsin. State'ni umumiy parent'ga ko'taring, biri o'zgarganda ikkinchisi avtomatik yangilanadigan qiling.

5. **Modal portal:** `createPortal` yordamida `document.body`'ga render bo'ladigan `Modal` component yozing. U `isOpen` prop'iga qarab ko'rinsin va "Yopish" tugmasi bo'lsin.

6. **Conditional rendering tuzog'i:** `{items.length && <List items={items} />}` kodi `items` bo'sh bo'lganda ekranda `0` chiqaradi. Sababini tushuntiring va tuzatilgan variantni yozing.

7. **Composition:** `title` prop va `children` qabul qiluvchi qayta ishlatiladigan `Card` component yozing. Uni uch xil turli kontent bilan ishlatib ko'rsating.

8. **List render:** Berilgan `users` massivini (`{ id, name, active }`) `<ul>` ichida render qiling; faqat `active === true` bo'lganlar ko'rinsin, har biriga to'g'ri `key` bering.

---

← [Frontend bo'limiga qaytish](./README.md)
