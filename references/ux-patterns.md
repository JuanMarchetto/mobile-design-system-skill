# Mobile UX Patterns

Touch interactions, gesture design, navigation patterns, mobile UI components, accessibility, and performance optimization for mobile interfaces.

---

## Touch Interactions

### Tap (Primary Interaction)

**Single Tap:**
- Primary action on buttons, links, list items
- Must provide immediate visual feedback (0-100ms delay)
- Minimum size: 48x48 pixels

```jsx
// Tap with visual feedback
function TapButton({ onPress, children }) {
  const [isPressed, setIsPressed] = useState(false);

  return (
    <button
      className={`tap-button ${isPressed ? 'pressed' : ''}`}
      onTouchStart={() => setIsPressed(true)}
      onTouchEnd={() => setIsPressed(false)}
      onTouchCancel={() => setIsPressed(false)}
      onClick={onPress}
    >
      {children}
    </button>
  );
}

// CSS
.tap-button {
  padding: 16px 24px;
  background: #007AFF;
  color: white;
  border: none;
  border-radius: 8px;
  font-size: 16px;
  min-height: 48px;
  transition: transform 0.1s, background 0.1s;
  -webkit-tap-highlight-color: transparent;
}

.tap-button.pressed {
  transform: scale(0.96);
  background: #0051D5;
}
```

**Double Tap:**
- Zoom in/out (maps, images)
- Like/favorite (social media)
- Use sparingly -- not discoverable

**Prevent double-tap zoom while allowing pinch zoom:**
```css
touch-action: manipulation;
```

### Swipe Gestures

**Horizontal Swipe:**
- Navigate between screens/pages
- Reveal actions (swipe-to-delete, swipe-to-archive)
- Dismiss cards/modals, switch tabs

```jsx
function SwipeableListItem({ children, onDelete, onArchive }) {
  const [touchStart, setTouchStart] = useState(null);
  const [touchEnd, setTouchEnd] = useState(null);
  const [translateX, setTranslateX] = useState(0);
  const minSwipeDistance = 50;

  const onTouchStart = (e) => {
    setTouchEnd(null);
    setTouchStart(e.targetTouches[0].clientX);
  };

  const onTouchMove = (e) => {
    setTouchEnd(e.targetTouches[0].clientX);
    const distance = touchStart - e.targetTouches[0].clientX;
    setTranslateX(-distance);
  };

  const onTouchEnd = () => {
    if (!touchStart || !touchEnd) return;
    const distance = touchStart - touchEnd;
    const isLeftSwipe = distance > minSwipeDistance;
    const isRightSwipe = distance < -minSwipeDistance;

    if (isLeftSwipe) {
      setTranslateX(-80); // Show actions
    } else {
      setTranslateX(0); // Snap back
    }
  };

  return (
    <div className="swipeable-item-container">
      <div className="swipe-actions">
        <button onClick={onArchive} className="archive-btn">Archive</button>
        <button onClick={onDelete} className="delete-btn">Delete</button>
      </div>
      <div
        className="swipeable-item"
        style={{ transform: `translateX(${translateX}px)` }}
        onTouchStart={onTouchStart}
        onTouchMove={onTouchMove}
        onTouchEnd={onTouchEnd}
      >
        {children}
      </div>
    </div>
  );
}
```

**Vertical Swipe:**
- Pull to refresh (downward swipe from top)
- Scroll content
- Dismiss bottom sheets/modals (downward swipe)

```jsx
function PullToRefresh({ onRefresh, children }) {
  const [pulling, setPulling] = useState(false);
  const [pullDistance, setPullDistance] = useState(0);
  const threshold = 80;

  const handleTouchStart = (e) => {
    if (window.scrollY === 0) setPulling(true);
  };

  const handleTouchMove = (e) => {
    if (pulling && window.scrollY === 0) {
      const distance = e.touches[0].clientY - e.touches[0].target.getBoundingClientRect().top;
      setPullDistance(Math.min(distance * 0.5, threshold * 1.5)); // Resistance
    }
  };

  const handleTouchEnd = () => {
    if (pullDistance >= threshold) onRefresh();
    setPulling(false);
    setPullDistance(0);
  };

  return (
    <div
      onTouchStart={handleTouchStart}
      onTouchMove={handleTouchMove}
      onTouchEnd={handleTouchEnd}
    >
      {pullDistance > 0 && (
        <div className="pull-indicator" style={{ height: pullDistance }}>
          {pullDistance >= threshold ? 'Release to refresh' : 'Pull to refresh'}
        </div>
      )}
      {children}
    </div>
  );
}
```

### Pinch and Spread (Zoom)

Used for image galleries, maps, PDF viewers, and any zoomable content.

```jsx
function PinchZoomImage({ src, alt }) {
  const [scale, setScale] = useState(1);
  const [lastScale, setLastScale] = useState(1);
  let lastDistance = null;

  const handleTouchMove = (e) => {
    if (e.touches.length === 2) {
      e.preventDefault();
      const touch1 = e.touches[0];
      const touch2 = e.touches[1];
      const distance = Math.hypot(
        touch1.clientX - touch2.clientX,
        touch1.clientY - touch2.clientY
      );

      if (lastDistance) {
        const newScale = lastScale * (distance / lastDistance);
        setScale(Math.max(1, Math.min(newScale, 4))); // Limit 1x to 4x
      }
      lastDistance = distance;
    }
  };

  const handleTouchEnd = () => {
    setLastScale(scale);
    lastDistance = null;
  };

  return (
    <div className="pinch-zoom-container"
      onTouchMove={handleTouchMove}
      onTouchEnd={handleTouchEnd}
    >
      <img src={src} alt={alt}
        style={{
          transform: `scale(${scale})`,
          transition: lastDistance ? 'none' : 'transform 0.2s',
        }}
      />
    </div>
  );
}
```

### Long Press

Used for context menus, item selection mode, drag-and-drop initiation.

```jsx
function useLongPress(callback, ms = 500) {
  const [startLongPress, setStartLongPress] = useState(false);

  useEffect(() => {
    let timerId;
    if (startLongPress) {
      timerId = setTimeout(callback, ms);
    }
    return () => clearTimeout(timerId);
  }, [startLongPress, callback, ms]);

  return {
    onTouchStart: () => setStartLongPress(true),
    onTouchEnd: () => setStartLongPress(false),
    onTouchMove: () => setStartLongPress(false),
  };
}
```

### Drag and Drop

```jsx
// React Native: Drag and Drop
import { PanResponder, Animated } from 'react-native';

function DraggableCard({ children }) {
  const pan = useRef(new Animated.ValueXY()).current;

  const panResponder = useRef(
    PanResponder.create({
      onStartShouldSetPanResponder: () => true,
      onPanResponderGrant: () => {
        pan.setOffset({ x: pan.x._value, y: pan.y._value });
      },
      onPanResponderMove: Animated.event(
        [null, { dx: pan.x, dy: pan.y }],
        { useNativeDriver: false }
      ),
      onPanResponderRelease: () => {
        pan.flattenOffset();
        Animated.spring(pan, {
          toValue: { x: 0, y: 0 },
          useNativeDriver: true,
        }).start();
      },
    })
  ).current;

  return (
    <Animated.View
      {...panResponder.panHandlers}
      style={{ transform: [{ translateX: pan.x }, { translateY: pan.y }] }}
    >
      {children}
    </Animated.View>
  );
}
```

---

## Navigation Patterns

### Tab Bar Navigation (Bottom)

- 3-5 tabs maximum
- Always show labels (don't rely on icons alone)
- Highlight active tab clearly
- Keep tabs visible at all times
- Most important section on the left (LTR languages)

```jsx
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';

const Tab = createBottomTabNavigator();

function AppNavigator() {
  return (
    <Tab.Navigator
      screenOptions={({ route }) => ({
        tabBarIcon: ({ focused, color, size }) => {
          let iconName;
          if (route.name === 'Home') iconName = focused ? 'home' : 'home-outline';
          else if (route.name === 'Search') iconName = focused ? 'search' : 'search-outline';
          else if (route.name === 'Profile') iconName = focused ? 'person' : 'person-outline';
          return <Icon name={iconName} size={size} color={color} />;
        },
        tabBarActiveTintColor: '#007AFF',
        tabBarInactiveTintColor: '#8E8E93',
        tabBarStyle: {
          height: 88,
          paddingBottom: 34, // iPhone home indicator
        },
      })}
    >
      <Tab.Screen name="Home" component={HomeScreen} />
      <Tab.Screen name="Search" component={SearchScreen} />
      <Tab.Screen name="Profile" component={ProfileScreen} />
    </Tab.Navigator>
  );
}
```

### Hamburger Menu (Drawer Navigation)

**When to use:**
- Secondary navigation
- Many navigation options (6+)
- Infrequently accessed features
- Settings and account options

**Avoid when:**
- Primary navigation is needed
- User needs quick access to all sections
- You have 5 or fewer main sections (use tabs instead)

```jsx
import { createDrawerNavigator } from '@react-navigation/drawer';

const Drawer = createDrawerNavigator();

function DrawerNavigator() {
  return (
    <Drawer.Navigator
      screenOptions={{
        drawerPosition: 'left',
        drawerType: 'slide',
        drawerStyle: { width: 280 },
        headerShown: true,
      }}
    >
      <Drawer.Screen name="Home" component={HomeScreen} />
      <Drawer.Screen name="Settings" component={SettingsScreen} />
    </Drawer.Navigator>
  );
}
```

### Bottom Sheets

```jsx
function BottomSheet({ isOpen, onClose, children }) {
  const [startY, setStartY] = useState(0);
  const [currentY, setCurrentY] = useState(0);

  const handleTouchStart = (e) => setStartY(e.touches[0].clientY);

  const handleTouchMove = (e) => {
    const delta = e.touches[0].clientY - startY;
    if (delta > 0) setCurrentY(delta); // Only allow downward drag
  };

  const handleTouchEnd = () => {
    if (currentY > 100) onClose(); // Threshold for closing
    setCurrentY(0);
  };

  if (!isOpen) return null;

  return (
    <>
      <div className="bottom-sheet-backdrop" onClick={onClose} />
      <div
        className="bottom-sheet"
        style={{ transform: `translateY(${currentY}px)` }}
        onTouchStart={handleTouchStart}
        onTouchMove={handleTouchMove}
        onTouchEnd={handleTouchEnd}
      >
        <div className="bottom-sheet-handle" />
        <div className="bottom-sheet-content">{children}</div>
      </div>
    </>
  );
}

// CSS
.bottom-sheet-backdrop {
  position: fixed; top: 0; left: 0; right: 0; bottom: 0;
  background: rgba(0, 0, 0, 0.5); z-index: 999;
}
.bottom-sheet {
  position: fixed; bottom: 0; left: 0; right: 0;
  background: white; border-radius: 20px 20px 0 0;
  padding: 16px; max-height: 80vh; z-index: 1000;
  transition: transform 0.3s;
}
.bottom-sheet-handle {
  width: 40px; height: 4px;
  background: #D1D1D6; border-radius: 2px;
  margin: 8px auto 16px;
}
```

### Stack Navigation

```jsx
import { createStackNavigator } from '@react-navigation/stack';

const Stack = createStackNavigator();

function StackNavigator() {
  return (
    <Stack.Navigator
      screenOptions={{
        headerStyle: { backgroundColor: '#007AFF' },
        headerTintColor: '#fff',
        headerTitleStyle: { fontWeight: 'bold' },
      }}
    >
      <Stack.Screen name="List" component={ListScreen} />
      <Stack.Screen name="Details" component={DetailsScreen} />
      <Stack.Screen name="Edit" component={EditScreen} />
    </Stack.Navigator>
  );
}
```

---

## Mobile UI Components

### Cards

```jsx
function Card({ image, title, subtitle, description, actions }) {
  return (
    <div className="card">
      {image && (
        <div className="card-media">
          <img src={image} alt={title} loading="lazy" />
        </div>
      )}
      <div className="card-content">
        <h3 className="card-title">{title}</h3>
        {subtitle && <p className="card-subtitle">{subtitle}</p>}
        <p className="card-description">{description}</p>
      </div>
      {actions && <div className="card-actions">{actions}</div>}
    </div>
  );
}

// CSS
.card {
  background: white; border-radius: 12px;
  overflow: hidden; box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  margin-bottom: 16px;
}
.card-media img { width: 100%; height: 200px; object-fit: cover; }
.card-content { padding: 16px; }
.card-title { font-size: 20px; font-weight: 600; margin: 0 0 8px 0; }
.card-subtitle { font-size: 14px; color: #666; margin: 0 0 8px 0; }
.card-actions { padding: 8px 16px 16px; display: flex; gap: 8px; }
```

### iOS-Style Lists

```jsx
function IOSList({ items, onItemPress }) {
  return (
    <div className="ios-list">
      {items.map((item, index) => (
        <div key={item.id} className="ios-list-item" onClick={() => onItemPress(item)}>
          {item.icon && <div className="ios-list-icon">{item.icon}</div>}
          <div className="ios-list-content">
            <div className="ios-list-title">{item.title}</div>
            {item.subtitle && <div className="ios-list-subtitle">{item.subtitle}</div>}
          </div>
          {item.badge && <div className="ios-list-badge">{item.badge}</div>}
          <div className="ios-list-chevron">&rsaquo;</div>
        </div>
      ))}
    </div>
  );
}

// CSS
.ios-list { background: white; border-radius: 12px; overflow: hidden; }
.ios-list-item {
  display: flex; align-items: center; padding: 12px 16px;
  min-height: 56px; border-bottom: 0.5px solid #E5E5EA;
  -webkit-tap-highlight-color: transparent;
}
.ios-list-item:active { background: #F2F2F7; }
.ios-list-item:last-child { border-bottom: none; }
```

### Mobile-Optimized Forms

```jsx
function MobileForm() {
  return (
    <form className="mobile-form">
      <div className="form-group">
        <label htmlFor="email">Email</label>
        <input id="email" type="email" inputMode="email" autoComplete="email"
          placeholder="you@example.com" />
      </div>
      <div className="form-group">
        <label htmlFor="phone">Phone</label>
        <input id="phone" type="tel" inputMode="tel" autoComplete="tel"
          placeholder="(555) 123-4567" />
      </div>
      <div className="form-group">
        <label htmlFor="amount">Amount</label>
        <input id="amount" type="number" inputMode="decimal" placeholder="0.00" />
      </div>
      <button type="submit" className="submit-button">Submit</button>
    </form>
  );
}

// CSS
.form-group input {
  width: 100%; height: 56px; padding: 16px;
  font-size: 16px; /* Prevents zoom on iOS */
  border: 2px solid #E5E5EA; border-radius: 12px;
  -webkit-appearance: none;
}
.form-group input:focus { outline: none; border-color: #007AFF; }
```

**Input Types for Mobile Keyboards:**
```html
<input type="email" inputmode="email">     <!-- Email keyboard -->
<input type="number" inputmode="numeric">  <!-- Numeric keyboard -->
<input type="number" inputmode="decimal">  <!-- Decimal keyboard -->
<input type="tel" inputmode="tel">         <!-- Telephone keyboard -->
<input type="url" inputmode="url">         <!-- URL keyboard -->
<input type="search" inputmode="search">   <!-- Search keyboard -->
```

### iOS-Style Action Sheet

```jsx
function ActionSheet({ isOpen, onClose, title, options }) {
  if (!isOpen) return null;

  return (
    <>
      <div className="action-sheet-backdrop" onClick={onClose} />
      <div className="action-sheet">
        {title && <div className="action-sheet-title">{title}</div>}
        <div className="action-sheet-options">
          {options.map((option, index) => (
            <button key={index}
              className={`action-sheet-option ${option.destructive ? 'destructive' : ''}`}
              onClick={() => { option.onPress(); onClose(); }}
            >
              {option.icon && <span className="option-icon">{option.icon}</span>}
              {option.label}
            </button>
          ))}
        </div>
        <button className="action-sheet-cancel" onClick={onClose}>Cancel</button>
      </div>
    </>
  );
}
```

---

## Platform Conventions

### iOS Human Interface Guidelines

- **Navigation Bar**: 44pt height (plus status bar), large title 52pt collapsible
- **Tab Bar**: 49pt height (plus safe area), 5 tabs max, badge notifications
- **Typography**: SF Pro, Dynamic Type required, 11pt-34pt
- **Colors**: System colors adapt to light/dark mode. Blue (#007AFF) tappable, Red (#FF3B30) destructive
- **Spacing**: 16pt minimum margins. Standard: 8pt, 16pt, 24pt, 32pt

### Material Design (Android)

- **App Bar**: 56dp height (64dp tablets), elevation 4dp
- **Bottom Navigation**: 56dp height, 3-5 destinations
- **FAB**: 56x56dp regular, 40x40dp mini, 16dp from edges
- **Typography**: Roboto, 12sp-96sp, line height 1.5x
- **Elevation**: 0dp flat, 1-8dp raised, 16-24dp modals
- **Spacing**: 4dp grid, keylines at 16dp and 72dp from edges

---

## Accessibility

### Touch Target Sizes

WCAG 2.1 Level AAA: 44x44px minimum, 48x48px recommended, 8px spacing between targets.

### Screen Reader Support

```jsx
// Semantic HTML for mobile navigation
function AccessibleMobileNav() {
  return (
    <nav role="navigation" aria-label="Main navigation">
      <ul>
        <li>
          <a href="/home" aria-current="page">
            <Icon name="home" aria-hidden="true" />
            <span>Home</span>
          </a>
        </li>
      </ul>
    </nav>
  );
}
```

### Color Contrast

WCAG AA: 4.5:1 for normal text, 3:1 for large text (18pt+), 3:1 for UI components.

```css
/* Good contrast */
.primary-button {
  background: #0066CC; color: #FFFFFF; /* 6.4:1 ratio */
}

/* Bad contrast - avoid */
.bad-button {
  background: #FFCC00; color: #FFFFFF; /* 1.4:1 ratio */
}
```

### Focus Indicators

```css
button:focus-visible {
  outline: 3px solid #007AFF;
  outline-offset: 2px;
}

input:focus-visible {
  border-color: #007AFF;
  box-shadow: 0 0 0 4px rgba(0, 122, 255, 0.2);
}
```

---

## Performance

### Image Optimization

```html
<!-- Responsive images -->
<img src="image-800w.jpg"
  srcset="image-400w.jpg 400w, image-800w.jpg 800w, image-1200w.jpg 1200w"
  sizes="(max-width: 480px) 100vw, (max-width: 768px) 50vw, 33vw"
  alt="Product image" loading="lazy" />

<!-- WebP with fallback -->
<picture>
  <source srcset="image.webp" type="image/webp">
  <source srcset="image.jpg" type="image/jpeg">
  <img src="image.jpg" alt="Fallback image">
</picture>
```

### Lazy Loading with Intersection Observer

```jsx
function LazyImage({ src, alt }) {
  const [isLoaded, setIsLoaded] = useState(false);
  const [isInView, setIsInView] = useState(false);
  const imgRef = useRef();

  useEffect(() => {
    const observer = new IntersectionObserver(
      ([entry]) => {
        if (entry.isIntersecting) {
          setIsInView(true);
          observer.disconnect();
        }
      },
      { rootMargin: '50px' }
    );
    if (imgRef.current) observer.observe(imgRef.current);
    return () => observer.disconnect();
  }, []);

  return (
    <div ref={imgRef} className="lazy-image-container">
      {!isLoaded && <div className="skeleton-loader" />}
      {isInView && (
        <img src={src} alt={alt}
          onLoad={() => setIsLoaded(true)}
          style={{ opacity: isLoaded ? 1 : 0 }}
        />
      )}
    </div>
  );
}
```

### Skeleton Screens

```jsx
function SkeletonCard() {
  return (
    <div className="skeleton-card">
      <div className="skeleton skeleton-image" />
      <div className="skeleton skeleton-title" />
      <div className="skeleton skeleton-text" />
      <div className="skeleton skeleton-text short" />
    </div>
  );
}

// CSS
.skeleton {
  background: linear-gradient(90deg, #f0f0f0 25%, #e0e0e0 50%, #f0f0f0 75%);
  background-size: 200% 100%;
  animation: loading 1.5s infinite;
}
@keyframes loading {
  0% { background-position: 200% 0; }
  100% { background-position: -200% 0; }
}
```

### Core Web Vitals for Mobile

- **LCP** (Largest Contentful Paint): < 2.5s
- **FID** (First Input Delay): < 100ms
- **CLS** (Cumulative Layout Shift): < 0.1

### PWA Service Worker

```javascript
const CACHE_NAME = 'mobile-app-v1';
const urlsToCache = ['/', '/styles/main.css', '/scripts/main.js'];

self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open(CACHE_NAME).then((cache) => cache.addAll(urlsToCache))
  );
});

self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.match(event.request)
      .then((response) => response || fetch(event.request))
  );
});
```

---

## Viewport and Safe Areas

### Viewport Meta Tag
```html
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=5, user-scalable=yes">

<!-- PWA with standalone mode -->
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
```

### Safe Areas (iPhone X+)
```css
.header {
  padding-top: env(safe-area-inset-top);
  padding-left: env(safe-area-inset-left);
  padding-right: env(safe-area-inset-right);
}
.footer {
  padding-bottom: env(safe-area-inset-bottom);
}
```

### Mobile-First Breakpoints
```css
/* Base: mobile (320px+) */
/* Small devices (480px+) */
@media (min-width: 480px) { }
/* Tablets (768px+) */
@media (min-width: 768px) { }
/* Laptops (1024px+) */
@media (min-width: 1024px) { }
/* Desktops (1280px+) */
@media (min-width: 1280px) { }
```

### Orientation-Specific Styles
```css
@media (orientation: portrait) {
  .container { flex-direction: column; }
}
@media (orientation: landscape) {
  .container { flex-direction: row; }
  .sidebar { width: 300px; }
}
/* Hide bottom nav when keyboard is open */
@media (max-height: 500px) {
  .bottom-nav { display: none; }
}
```

### Container Queries
```css
.card-container { container-type: inline-size; }

@container (min-width: 400px) {
  .card { display: grid; grid-template-columns: 1fr 2fr; }
}
```

---

## Touch Target Quick Reference

| Platform | Minimum Size | Optimal Size | Spacing |
|----------|-------------|--------------|---------|
| iOS | 44x44 pt | 56x56 pt | 8pt |
| Android | 48x48 dp | 56x56 dp | 8dp |
| Web (WCAG) | 44x44 px | 48x48 px | 8px |

## Thumb Zones

1. **Easy Zone** (bottom center): Primary actions, CTAs, tab bars
2. **Stretch Zone** (middle): Content, forms, secondary actions
3. **Difficult Zone** (top corners): Destructive actions, infrequent settings

## Performance Checklist

```
[ ] Images use lazy loading
[ ] Responsive images with srcset
[ ] WebP format with fallback
[ ] Critical CSS inlined
[ ] JavaScript code split
[ ] Service worker for caching
[ ] Skeleton screens for loading
[ ] Touch interactions < 100ms
[ ] LCP < 2.5 seconds
[ ] CLS < 0.1
```

## Accessibility Checklist

```
[ ] Touch targets >= 48x48 pixels
[ ] Color contrast >= 4.5:1
[ ] Focus indicators visible
[ ] Screen reader labels present
[ ] Keyboard navigation works
[ ] Zoom up to 200% supported
[ ] Orientation changes handled
[ ] Form inputs labeled properly
[ ] Error messages clear
[ ] Dynamic Type supported
```
