```
(function (global, factory) {
  typeof exports === 'object' && typeof module !== 'undefined' ? factory(exports) :
  typeof define === 'function' && define.amd ? define(['exports'], factory) :
  (global = typeof globalThis !== 'undefined' ? globalThis : global || self, factory(global.Forkpack = {}));
})(this, (function (exports) {
  'use strict';

  /*! Forkpack Discount Banner Widget v2.1.0 | MIT License | forkpack.ru */

  // ==================== UMD WRAPPER + TYPES ====================
  var VERSION = '2.1.0';
  
  var DEFAULT_CONFIG = {
    discount: 25,
    expires: '',
    ctaText: 'КУПИТЬ СО СКИДКОЙ',
    ctaUrl: '/',
    position: 'top',
    theme: 'auto',
    target: null
  };

  // ==================== COUNTDOWN CLASS ====================
  var CountdownTimer = (function () {
    function CountdownTimer(expiresISO) {
      this.expires = new Date(expiresISO);
      this.interval = null;
      this.callbacks = [];
    }

    CountdownTimer.prototype.start = function (updateCallback) {
      if (updateCallback) this.callbacks.push(updateCallback);
      
      var self = this;
      var update = function () {
        var now = new Date();
        var diff = self.expires - now;
        
        if (diff <= 0) {
          self.stop();
          self.notify('00:00:00:00', true);
          return;
        }
        
        var days = Math.floor(diff / (1000 * 60 * 60 * 24));
        var hours = Math.floor((diff % (1000 * 60 * 60 * 24)) / (1000 * 60 * 60));
        var minutes = Math.floor((diff % (1000 * 60 * 60)) / (1000 * 60));
        var seconds = Math.floor((diff % (1000 * 60)) / 1000);
        
        var formatted = days + 'д ' + 
          String(hours).padStart(2, '0') + 'ч ' + 
          String(minutes).padStart(2, '0') + 'м ' + 
          String(seconds).padStart(2, '0') + 'с';
        
        var ariaText = days + ' дней ' + hours + ' часов ' + 
          minutes + ' минут ' + seconds + ' секунд';
        
        self.notify(formatted, false, ariaText);
      };
      
      update();
      this.interval = setInterval(update, 1000);
    };

    CountdownTimer.prototype.stop = function () {
      if (this.interval) {
        clearInterval(this.interval);
        this.interval = null;
      }
    };

    CountdownTimer.prototype.notify = function (text, expired, ariaText) {
      this.callbacks.forEach(function (cb) {
        cb(text, expired, ariaText);
      });
    };

    CountdownTimer.prototype.destroy = function () {
      this.stop();
      this.callbacks = [];
    };

    return CountdownTimer;
  })();

  // ==================== DOM UTILS ====================
  var DomUtils = {
    createElement: function (tag, className, attrs) {
      var el = document.createElement(tag);
      if (className) el.className = className;
      if (attrs) {
        Object.keys(attrs).forEach(function (key) {
          el.setAttribute(key, attrs[key]);
        });
      }
      return el;
    },

    injectStyles: function (css) {
      if (document.querySelector('style[data-fp-discount]')) return;
      
      var style = document.createElement('style');
      style.setAttribute('data-fp-discount', '');
      style.textContent = css;
      document.head.appendChild(style);
    },

    getTargetElement: function (target) {
      if (!target) return document.body;
      if (typeof target === 'string') return document.querySelector(target);
      if (target instanceof Element) return target;
      return document.body;
    },

    generateId: function () {
      return 'fp-discount-' + Date.now() + '-' + Math.random().toString(36).substr(2, 9);
    }
  };

  // ==================== THEME DETECTOR ====================
  var ThemeDetector = (function () {
    var THEMES = {
      light: { bg: '#fef7f2', text: '#1f2937', primary: '#ff6b35' },
      dark: { bg: '#1f2937', text: '#f3f4f6', primary: '#ff8c5a' }
    };

    function getSystemTheme() {
      return window.matchMedia && window.matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light';
    }

    function detectTheme(themePreference) {
      if (themePreference === 'dark' || themePreference === 'light') {
        return themePreference;
      }
      return getSystemTheme();
    }

    function applyThemeVars(container, theme) {
      var colors = THEMES[theme];
      container.style.setProperty('--fp-bg', colors.bg);
      container.style.setProperty('--fp-text', colors.text);
      container.style.setProperty('--fp-primary', colors.primary);
    }

    function watchSystemTheme(container, currentTheme) {
      if (!window.matchMedia) return null;
      
      var mediaQuery = window.matchMedia('(prefers-color-scheme: dark)');
      var handler = function (e) {
        var newTheme = e.matches ? 'dark' : 'light';
        if (currentTheme === 'auto') {
          applyThemeVars(container, newTheme);
        }
      };
      
      mediaQuery.addEventListener('change', handler);
      return handler;
    }

    return {
      detect: detectTheme,
      apply: applyThemeVars,
      watch: watchSystemTheme
    };
  })();

  // ==================== ANIMATION ENGINE ====================
  var AnimationEngine = {
    injectBaseStyles: function () {
      var css = `
        .fp-discount-banner {
          --fp-primary: #ff6b35;
          --fp-bg: #fef7f2;
          --fp-text: #1f2937;
          --fp-accent: #059669;
          --fp-shadow: 0 20px 40px rgba(0,0,0,0.1);
          --fp-radius: 12px;
          --fp-ease: cubic-bezier(0.4, 0, 0.2, 1);
          
          box-sizing: border-box;
          font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, sans-serif;
          position: fixed;
          left: 0;
          right: 0;
          z-index: 9999;
          padding: 16px;
          background: var(--fp-bg);
          color: var(--fp-text);
          box-shadow: var(--fp-shadow);
          transition: transform 0.8s var(--fp-ease), opacity 0.3s;
          display: flex;
          align-items: center;
          justify-content: center;
          gap: 16px;
          flex-wrap: wrap;
          text-align: center;
          line-height: 1.5;
          border: 1px solid rgba(255, 107, 53, 0.1);
        }
        
        .fp-discount-banner.top {
          top: 0;
          border-bottom: 2px solid var(--fp-primary);
          animation: fp-slide-down 0.8s var(--fp-ease);
        }
        
        .fp-discount-banner.bottom {
          bottom: 0;
          border-top: 2px solid var(--fp-primary);
          animation: fp-slide-up 0.8s var(--fp-ease);
        }
        
        .fp-discount-content {
          display: flex;
          align-items: center;
          gap: 8px 24px;
          flex-wrap: wrap;
          justify-content: center;
          max-width: 1200px;
        }
        
        .fp-discount-badge {
          background: linear-gradient(135deg, var(--fp-primary) 0%, #ff8c5a 100%);
          color: white;
          font-weight: 900;
          font-size: 2.5em;
          padding: 8px 20px;
          border-radius: var(--fp-radius);
          display: inline-flex;
          align-items: center;
          justify-content: center;
          min-width: 120px;
          text-shadow: 0 2px 4px rgba(0,0,0,0.2);
          animation: fp-pulse 2s infinite;
        }
        
        .fp-discount-text {
          font-size: 1.2em;
          font-weight: 600;
          display: flex;
          align-items: center;
          gap: 8px;
          flex-wrap: wrap;
        }
        
        .fp-timer {
          background: rgba(255, 107, 53, 0.1);
          padding: 8px 16px;
          border-radius: var(--fp-radius);
          font-family: 'Courier New', monospace;
          font-weight: 700;
          font-size: 1.1em;
          border: 1px dashed var(--fp-primary);
          min-width: 280px;
        }
        
        .fp-timer[aria-label*="0 дней 0 часов"] {
          animation: fp-pulse-fast 0.5s infinite;
        }
        
        .fp-cta-button {
          background: var(--fp-primary);
          color: white;
          border: none;
          padding: 12px 32px;
          border-radius: var(--fp-radius);
          font-weight: 800;
          font-size: 1em;
          cursor: pointer;
          transition: all 0.3s var(--fp-ease);
          text-transform: uppercase;
          letter-spacing: 0.5px;
          min-width: 180px;
        }
        
        .fp-cta-button:hover,
        .fp-cta-button:focus {
          background: #e55a2b;
          transform: translateY(-2px);
          box-shadow: 0 8px 20px rgba(255, 107, 53, 0.4);
          outline: 2px solid white;
          outline-offset: 2px;
        }
        
        .fp-cta-button:active {
          transform: translateY(0);
        }
        
        .fp-close-button {
          background: transparent;
          border: none;
          color: var(--fp-text);
          opacity: 0.5;
          cursor: pointer;
          font-size: 1.5em;
          width: 32px;
          height: 32px;
          display: flex;
          align-items: center;
          justify-content: center;
          border-radius: 50%;
          transition: all 0.3s;
          position: absolute;
          top: 8px;
          right: 8px;
        }
        
        .fp-close-button:hover {
          opacity: 1;
          background: rgba(0,0,0,0.05);
        }
        
        @keyframes fp-slide-down {
          from {
            transform: translateY(-100%);
            opacity: 0;
          }
          to {
            transform: translateY(0);
            opacity: 1;
          }
        }
        
        @keyframes fp-slide-up {
          from {
            transform: translateY(100%);
            opacity: 0;
          }
          to {
            transform: translateY(0);
            opacity: 1;
          }
        }
        
        @keyframes fp-pulse {
          0%, 100% {
            transform: scale(1);
            box-shadow: 0 4px 12px rgba(255, 107, 53, 0.3);
          }
          50% {
            transform: scale(1.02);
            box-shadow: 0 6px 20px rgba(255, 107, 53, 0.5);
          }
        }
        
        @keyframes fp-pulse-fast {
          0%, 100% {
            background: rgba(220, 38, 38, 0.1);
            border-color: #dc2626;
          }
          50% {
            background: rgba(220, 38, 38, 0.3);
            border-color: #ef4444;
          }
        }
        
        /* Responsive */
        @media (max-width: 768px) {
          .fp-discount-banner {
            padding: 12px;
            gap: 12px;
          }
          
          .fp-discount-content {
            gap: 12px;
          }
          
          .fp-discount-badge {
            font-size: 2em;
            min-width: 100px;
            padding: 6px 16px;
          }
          
          .fp-timer {
            min-width: 240px;
            font-size: 1em;
            padding: 6px 12px;
          }
          
          .fp-cta-button {
            padding: 10px 24px;
            min-width: 160px;
          }
        }
        
        @media (max-width: 480px) {
          .fp-discount-banner {
            padding: 8px;
          }
          
          .fp-discount-content {
            flex-direction: column;
            gap: 8px;
          }
          
          .fp-discount-text {
            font-size: 1em;
            flex-direction: column;
          }
          
          .fp-timer {
            min-width: 200px;
            font-size: 0.9em;
          }
          
          .fp-cta-button {
            width: 100%;
            max-width: 280px;
          }
        }
        
        @media (min-width: 1920px) {
          .fp-discount-banner {
            padding: 20px;
          }
          
          .fp-discount-badge {
            font-size: 3em;
          }
        }
        
        /* Accessibility */
        .fp-discount-banner *:focus {
          outline: 2px solid var(--fp-primary);
          outline-offset: 2px;
        }
        
        .sr-only {
          position: absolute;
          width: 1px;
          height: 1px;
          padding: 0;
          margin: -1px;
          overflow: hidden;
          clip: rect(0, 0, 0, 0);
          white-space: nowrap;
          border: 0;
        }
      `;
      
      DomUtils.injectStyles(css);
    }
  };

  // ==================== DISCOUNT BANNER CLASS ====================
  var DiscountBanner = (function () {
    function DiscountBanner(config) {
      this.config = Object.assign({}, DEFAULT_CONFIG, config);
      this.id = DomUtils.generateId();
      this.container = null;
      this.timer = null;
      this.themeHandler = null;
      this.isDestroyed = false;
      this.validateConfig();
    }

    DiscountBanner.prototype.validateConfig = function () {
      if (this.config.discount < 1 || this.config.discount > 99) {
        console.warn('Forkpack: discount must be between 1-99%, using default 25%');
        this.config.discount = 25;
      }
      
      if (this.config.expires) {
        var expiresDate = new Date(this.config.expires);
        if (isNaN(expiresDate.getTime())) {
          console.warn('Forkpack: invalid expires date, ignoring countdown');
          this.config.expires = '';
        }
      }
    };

    DiscountBanner.prototype.render = function () {
      if (this.isDestroyed) return null;
      
      AnimationEngine.injectBaseStyles();
      
      var target = DomUtils.getTargetElement(this.config.target);
      var currentTheme = ThemeDetector.detect(this.config.theme);
      
      this.container = DomUtils.createElement('div', 
        'fp-discount-banner ' + this.config.position,
        {
          'id': this.id,
          'role': 'banner',
          'aria-label': 'Баннер скидки ' + this.config.discount + '%'
        }
      );
      
      ThemeDetector.apply(this.container, currentTheme);
      
      if (this.config.theme === 'auto') {
        this.themeHandler = ThemeDetector.watch(this.container, this.config.theme);
      }
      
      var content = DomUtils.createElement('div', 'fp-discount-content');
      
      // Discount badge
      var badge = DomUtils.createElement('div', 'fp-discount-badge');
      badge.textContent = '-' + this.config.discount + '%';
      badge.setAttribute('aria-hidden', 'true');
      
      // Text content
      var text = DomUtils.createElement('div', 'fp-discount-text');
      var textSpan = DomUtils.createElement('span');
      textSpan.textContent = 'Скидка действует:';
      
      var timer = DomUtils.createElement('div', 'fp-timer', {
        'role': 'timer',
        'aria-live': 'polite',
        'aria-atomic': 'true'
      });
      
      text.appendChild(textSpan);
      text.appendChild(timer);
      
      // CTA Button
      var cta = DomUtils.createElement('button', 'fp-cta-button', {
        'type': 'button',
        'tabindex': '0'
      });
      cta.textContent = this.config.ctaText;
      
      // Close button
      var closeBtn = DomUtils.createElement('button', 'fp-close-button', {
        'type': 'button',
        'aria-label': 'Закрыть баннер'
      });
      closeBtn.innerHTML = '&times;';
      
      // Screen reader announcement
      var srAnnouncement = DomUtils.createElement('div', 'sr-only', {
        'aria-live': 'assertive',
        'aria-atomic': 'true'
      });
      
      content.appendChild(badge);
      content.appendChild(text);
      content.appendChild(cta);
      this.container.appendChild(content);
      this.container.appendChild(closeBtn);
      this.container.appendChild(srAnnouncement);
      
      target.appendChild(this.container);
      
      // Initialize countdown if expires date provided
      if (this.config.expires) {
        this.initCountdown(timer, srAnnouncement);
      } else {
        timer.textContent = 'Ограниченное предложение';
        timer.setAttribute('aria-label', 'Ограниченное предложение');
      }
      
      // Add event listeners
      this.setupEventListeners(cta, closeBtn);
      
      // Announce to screen readers
      this.announceToScreenReader('Скидка ' + this.config.discount + ' процентов доступна. ' + 
        (this.config.expires ? 'Предложение ограничено по времени.' : ''));
      
      return this.container;
    };

    DiscountBanner.prototype.initCountdown = function (timerElement, srElement) {
      var self = this;
      this.timer = new CountdownTimer(this.config.expires);
      
      this.timer.start(function (text, expired, ariaText) {
        if (self.isDestroyed) return;
        
        timerElement.textContent = text;
        timerElement.setAttribute('aria-label', 'Осталось времени: ' + ariaText);
        
        if (expired) {
          timerElement.textContent = 'Время истекло!';
          timerElement.setAttribute('aria-label', 'Время действия скидки истекло');
          self.announceToScreenReader('Время действия скидки истекло');
        }
        
        // Update screen reader announcement every 30 seconds
        var now = new Date();
        if (now.getSeconds() % 30 === 0) {
          srElement.textContent = 'Скидка ' + self.config.discount + ' процентов. Осталось: ' + ariaText;
        }
      });
    };

    DiscountBanner.prototype.setupEventListeners = function (ctaButton, closeButton) {
      var self = this;
      
      // CTA click
      ctaButton.addEventListener('click', function () {
        if (self.config.ctaUrl) {
          window.location.href = self.config.ctaUrl;
        }
      });
      
      // CTA keyboard support
      ctaButton.addEventListener('keydown', function (e) {
        if (e.key === 'Enter' || e.key === ' ') {
          e.preventDefault();
          if (self.config.ctaUrl) {
            window.location.href = self.config.ctaUrl;
          }
        }
      });
      
      // Close button
      closeButton.addEventListener('click', function () {
        self.destroy();
      });
      
      // Escape key support
      document.addEventListener('keydown', function (e) {
        if (e.key === 'Escape' && document.activeElement.closest('.fp-discount-banner')) {
          self.destroy();
        }
      }, { once: false });
      
      // Prevent memory leaks on page unload
      window.addEventListener('beforeunload', function () {
        self.destroy();
      });
    };

    DiscountBanner.prototype.announceToScreenReader = function (message) {
      var announcement = DomUtils.createElement('div', 'sr-only', {
        'aria-live': 'assertive',
        'aria-atomic': 'true'
      });
      announcement.textContent = message;
      document.body.appendChild(announcement);
      
      setTimeout(function () {
        if (announcement.parentNode) {
          announcement.parentNode.removeChild(announcement);
        }
      }, 1000);
    };

    DiscountBanner.prototype.destroy = function () {
      if (this.isDestroyed) return;
      
      this.isDestroyed = true;
      
      if (this.timer) {
        this.timer.destroy();
        this.timer = null;
      }
      
      if (this.themeHandler && window.matchMedia) {
        var mediaQuery = window.matchMedia('(prefers-color-scheme: dark)');
        mediaQuery.removeEventListener('change', this.themeHandler);
        this.themeHandler = null;
      }
      
      if (this.container && this.container.parentNode) {
        this.container.parentNode.removeChild(this.container);
      }
      
      // Clean up global event listeners
      document.removeEventListener('keydown', this.handleEscape);
    };

    return DiscountBanner;
  })();

  // ==================== AUTO-INIT + DATA ATTRIBUTES PARSER ====================
  var AutoInit = {
    init: function () {
      var widgets = document.querySelectorAll('[data-fp-widget="discount"]');
      
      widgets.forEach(function (element) {
        if (element.hasAttribute('data-fp-processed')) return;
        
        var config = {
          target: element,
          discount: parseInt(element.getAttribute('data-fp-discount')) || DEFAULT_CONFIG.discount,
          expires: element.getAttribute('data-fp-expires') || DEFAULT_CONFIG.expires,
          ctaText: element.getAttribute('data-fp-cta-text') || DEFAULT_CONFIG.ctaText,
          ctaUrl: element.getAttribute('data-fp-cta-url') || DEFAULT_CONFIG.ctaUrl,
          position: element.getAttribute('data-fp-position') || DEFAULT_CONFIG.position,
          theme: element.getAttribute('data-fp-theme') || DEFAULT_CONFIG.theme
        };
        
        var banner = new DiscountBanner(config);
        banner.render();
        
        element.setAttribute('data-fp-processed', 'true');
      });
    }
  };

  // ==================== WINDOW.FORKPACK API ====================
  var ForkpackAPI = {
    VERSION: VERSION,
    
    renderDiscountBanner: function (props) {
      try {
        var banner = new DiscountBanner(props);
        return banner.render();
      } catch (error) {
        console.error('Forkpack: Failed to render banner', error);
        return null;
      }
    },
    
    destroyAllBanners: function () {
      var banners = document.querySelectorAll('.fp-discount-banner');
      banners.forEach(function (banner) {
        banner.parentNode.removeChild(banner);
      });
      
      var styles = document.querySelectorAll('style[data-fp-discount]');
      styles.forEach(function (style) {
        style.parentNode.removeChild(style);
      });
    },
    
    getActiveBanners: function () {
      return Array.from(document.querySelectorAll('.fp-discount-banner')).map(function (banner) {
        return banner.id;
      });
    }
  };

  // ==================== INITIALIZATION ====================
  if (typeof document !== 'undefined') {
    if (document.readyState === 'loading') {
      document.addEventListener('DOMContentLoaded', function () {
        AutoInit.init();
      });
    } else {
      AutoInit.init();
    }
  }

  // ==================== EXPORTS ====================
  exports.renderDiscountBanner = ForkpackAPI.renderDiscountBanner;
  exports.destroyAllBanners = ForkpackAPI.destroyAllBanners;
  exports.getActiveBanners = ForkpackAPI.getActiveBanners;
  exports.VERSION = VERSION;

  Object.defineProperty(exports, '__esModule', { value: true });

}));
```

```
<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Forkpack Discount Banner Demo</title>
  <style>
    body {
      font-family: system-ui, -apple-system, sans-serif;
      max-width: 1200px;
      margin: 0 auto;
      padding: 20px;
      min-height: 200vh;
    }
    .demo-section {
      margin: 40px 0;
      padding: 20px;
      background: #f8fafc;
      border-radius: 12px;
    }
    code {
      background: #1e293b;
      color: #e2e8f0;
      padding: 2px 6px;
      border-radius: 4px;
      font-family: 'Courier New', monospace;
    }
  </style>
</head>
<body>
  <h1>Forkpack Discount Banner Widget v2.1.0</h1>
  
  <div class="demo-section">
    <h2>Авто-рендер через data-* атрибуты</h2>
    <div data-fp-widget="discount" 
         data-fp-discount="35" 
         data-fp-expires="2026-01-25T23:59:59Z"
         data-fp-cta-text="ЗАБРОНИРОВАТЬ"
         data-fp-position="top">
    </div>
    
    <p>Прокрутите вниз чтобы увидеть второй баннер</p>
  </div>
  
  <div class="demo-section">
    <h2>Программный рендер через JS API</h2>
    <div id="custom-banner-target"></div>
    <button onclick="renderCustomBanner()">Показать кастомный баннер</button>
    <button onclick="Forkpack.destroyAllBanners()">Удалить все баннеры</button>
  </div>
  
  <div class="demo-section">
    <h2>Второй авто-баннер (нижний)</h2>
    <div data-fp-widget="discount" 
         data-fp-discount="50" 
         data-fp-expires="2026-02-01T12:00:00Z"
         data-fp-cta-text="ПОЛУЧИТЬ ПРЕДЛОЖЕНИЕ"
         data-fp-position="bottom"
         data-fp-theme="dark">
    </div>
  </div>

  <!-- Подключаем виджет -->
  <script src="https://forkpack.ru/cdn/fp-discount.umd.js"></script>
  
  <script>
    function renderCustomBanner() {
      Forkpack.renderDiscountBanner({
        target: '#custom-banner-target',
        discount: 15,
        expires: '2026-01-20T18:30:00Z',
        ctaText: 'КУПИТЬ СО СКИДКОЙ',
        ctaUrl: '#',
        position: 'top',
        theme: 'auto'
      });
    }
    
    // Проверяем API
    console.log('Forkpack Discount Banner v' + Forkpack.VERSION);
    console.log('Active banners:', Forkpack.getActiveBanners());
  </script>
</body>
</html>
```
