# 🎯 Discount Banner Widget

```
🧠 КОНТЕКСТ
CDN пакет embed-виджетов для сайтов. 
Подключается через `<script src="https://forkpack.ru/cdn/fp.umd.js">`.
Виджет рендерится в DOM автоматически по apiKey.

🎯 ЗАДАЧА
Создать JavaScript embed-виджет баннера скидок:
- UMD сборка для CDN
- API: discount, expires, ctaText, position
- CSS vars для кастомизации
- Анимация Framer Motion
- Responsive mobile-first

📋 ОСНОВНОЙ ПРОМПТ Discount Banner Widget (Pure JS Module)

Создай самодостаточный UMD модуль Discount Banner Widget v2.1.0 для Forkpack.ru CDN на чистом JavaScript (без React/Framer).

🎯 Требования
✅ 10000 строк максимум (один файл fp-discount.umd.js)
✅ Zero external dependencies (CDN React/Framer НЕТ)
✅ UMD формат: <script src="forkpack.ru/cdn/fp-discount.umd.js"></script>
✅ Авто-рендер по data-fp-* атрибутам
✅ JS API: Forkpack.renderDiscountBanner(props)
✅ Bundle <10KB (gzip)

🔌 Подключение
<!-- 1. CDN -->
<script src="https://forkpack.ru/cdn/fp-discount.umd.js"></script>

<!-- 2. Авто-рендер -->
<div data-fp-widget="discount" 
     data-fp-discount="35" 
     data-fp-expires="2026-01-25T23:59:59Z">
</div>

<!-- 3. JS API -->
<script>
Forkpack.renderDiscountBanner({
  target: '#banner',
  discount: 35,
  expires: '2026-01-25T23:59:59Z',
  ctaText: 'КУПИТЬ',
  position: 'top'
});
</script>

📐 API Параметры

discount: number     // 5-99%
expires: string      // ISO Date "2026-01-25T23:59:59Z"
ctaText: string      // "КУПИТЬ СО СКИДКОЙ"
ctaUrl: string       // "/" 
position: "top|bottom"
theme: "light|dark|auto"
target: string|Element // "#id" или DOM element

🎨 Features (нативный JS)
⏱️ Таймер: setInterval(1000ms) → Д:Ч:М:С, auto-stop
✨ Анимации: CSS keyframes (slide-in 0.8s, pulse <60s)
🌙 Темы: matchMedia + CSS vars
♿ ARIA: role="timer", aria-live="polite", tabindex=0
📱 Responsive: CSS media queries 320-1920px
🧹 Cleanup: clearInterval + removeEventListener


🎨 CSS Vars (встроены)
--fp-primary: #ff6b35;
--fp-bg: #fef7f2;
--fp-text: #1f2937;
--fp-accent: #059669;
--fp-shadow: 0 20px 40px rgba(0,0,0,0.1);
--fp-radius: 12px;

📋 Структура кода (250 строк)
1-30:    UMD wrapper + types
31-70:   Countdown class + DOM utils
71-120:  ThemeDetector + Animation engine
121-180: DiscountBanner class (render + events)
181-220: Auto-init + data-* parser
221-250: window.Forkpack API + exports

✅ Критерии приемки
✅ Lighthouse: Performance 100, A11y 100
✅ Bundle: <10KB gzipped  
✅ Browsers: Chrome 110+, Firefox 120+, Safari 17+
✅ Keyboard: Tab → CTA → Enter
✅ Screenreader: "Скидка 35% -  2д 14ч 23м"
✅ Memory: no leaks (clearInterval проверено)

🚀 Demo HTML
<!DOCTYPE html>
<html>
<body>
  <div data-fp-widget="discount" data-fp-discount="25"></div>
  <script src="fp-discount.umd.js"></script>
</body>
</html>
```

Результат: **готовый к деплою на CDN** модуль (копипаст в forkpack.ru).

