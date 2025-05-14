# E-Menu Template System

*A modular digital restaurant menu system with support for multiple visual templates, languages, and currencies.*

**Test Point: [https://qr-api.e-menu.ai/](https://qr-api.e-menu.ai/)**

## 🌟 Key Features

- Multiple visual templates with consistent data structure
- Dynamic theme colors and styling
- Multi-language support with RTL capabilities
- Multiple currency support
- Favorites/cart functionality
- Performance optimized with lazy loading and code splitting
- Responsive design for all device sizes

## 🛠️ Tech Stack

- **Next.js**: Server-side rendering and routing
- **React**: Component-based UI development
- **Redux Toolkit & RTK Query**: State management and API calls
- **TailwindCSS**: Utility-first styling
- **Context API**: Shared state across components
- **GSAP**: Smooth animations

## 🏗️ System Architecture

The E-Menu application follows a modular architecture where templates are pluggable components that receive standardized data through props and context.

### Directory Structure

```
├── components/
│   ├── layouts/             # Layout components
│   ├── pages/               # Page-specific components
│   ├── sections/            # Reusable section components
│   ├── shared/              # Shared UI components
│   │   ├── cards/           # Card components for items
│   │   ├── modal/           # Modal components
│   │   └── others/          # Other utility components
│   └── templates/           # Template components (add yours here)
│       └── KardiTemplate.jsx # Example template
├── constants/               # Global constants and helpers
├── context/                 # React context providers
│   ├── FavoritesContext.js  # Cart/favorites state
│   └── GlobalContext.jsx    # App-wide state
├── data/                    # Data management
│   └── index.js             # Template registration
├── pages/                   # Next.js pages
│   ├── [slug].jsx           # Dynamic page for rendering templates
│   └── index.js             # Documentation page
├── services/                # API services
└── styles/                  # Global styles
```

### Data Flow

1. User navigates to `/{slug}` (restaurant identifier)
2. The `[slug].jsx` page fetches business data including theme
3. Based on the theme, the appropriate template is loaded
4. Business data, categories, and items are passed to the template
5. The template renders the menu with the provided data

### Template Loading

Templates are loaded dynamically to enable code splitting:

```javascript
// In data/index.js
export const templatesMap = {
  kardi: () => import("@/components/templates/KardiTemplate"),
  // Add your template here
  newtemplate: () => import("@/components/templates/NewTemplate"),
};

// In pages/[slug].jsx
useEffect(() => {
  if (!businessInfoData?.data?.theme) return;

  const theme = businessInfoData.data.theme.toLowerCase();
  if (theme && templatesMap[theme]) {
    templatesMap[theme]().then((module) => {
      setLazyTemplate(() => module.default);
    });
  }
}, [businessInfoData]);
```

## 📝 Creating Templates

### 1. Create Template Component

Create a new file in the `components/templates/` directory:

```javascript
// components/templates/MyTemplate.jsx
import React, { useEffect, useState, memo, lazy, Suspense } from "react";
import Header from "@/components/shared/Header";
import { useFavorites } from "@/context/FavoritesContext";
import { useGlobalContext } from "@/context/GlobalContext";

const MyTemplate = ({
  templateData,
  categoriesData,
  itemsData,
  itemsLoading,
  onHomeButtonClick,
}) => {
  // Context and state setup
  const { favorites, setFavorites, count } = useFavorites();
  const { selectedLang, selectedCurreny } = useGlobalContext();
  const [isGrid, setIsGrid] = useState(true);

  return (
    <>
      {/* Header - using shared component */}
      <Header
        templateData={templateData}
        onHomeButtonClicked={onHomeButtonClick}
      />

      {/* Your custom template content */}
      <main className="p-4">
        {/* Implement your menu display here */}
      </main>

      {/* Optional cart button */}
      {count > 0 && (
        <button
          className="fixed bottom-6 right-6 bg-primary text-white p-4 rounded-full shadow-lg"
          onClick={() => /* handle cart */}
        >
          Cart ({count})
        </button>
      )}
    </>
  );
};

// Memoize for performance
export default memo(MyTemplate);
```

### 2. Register Your Template

Add your template to `data/index.js`:

```javascript
// data/index.js
export const templatesMap = {
  kardi: () => import("@/components/templates/KardiTemplate"),
  // Add your template
  mytemplate: () => import("@/components/templates/MyTemplate"),
};
```

### 3. Required Props

Your template should accept these props:

- `templateData`: Business information (colors, name, languages, currencies)
- `categoriesData`: Menu categories
- `itemsData`: Menu items/foods
- `itemsLoading`: Loading state for items
- `onHomeButtonClick`: Function to handle home button click

### 4. Required Functionality

Your template should implement:

- Display of menu items with appropriate formatting
- Category filtering (using `selectedCategory` from context)
- Favorites/cart management (using `FavoritesContext`)
- Language support (using `selectedLang` from context)
- Currency support (using `selectedCurreny` from context)

## 🌐 Context & State Management

### GlobalContext

For app-wide state including theme, language, and currency:

```javascript
// In your template
import {useGlobalContext} from "@/context/GlobalContext";

const MyTemplate = () => {
  const {
    colors, // Theme colors
    selectedLang, // Selected language
    setSelectedLang, // Set selected language
    selectedCurreny, // Selected currency
    setSelectedCurreny, // Set selected currency
    selectedCategory, // Selected category
    setSelectedCategory, // Set selected category
    subscriptionDetails, // Subscription features
    translationsData, // Translation strings
  } = useGlobalContext();

  // ...
};
```

### FavoritesContext

For cart/favorites functionality:

```javascript
// In your template
import {useFavorites} from "@/context/FavoritesContext";

const MyTemplate = () => {
  const {
    favorites, // Array of favorited/cart items
    setFavorites, // Function to update favorites
    count, // Number of items in favorites/cart
  } = useFavorites();

  // Add to favorites example
  const addToFavorites = (item) => {
    setFavorites((prev) => [
      ...prev,
      {
        id: item.id,
        name: item,
        discount: item?.discount,
        price: item?.prices,
        img: item.thumbnail,
        quantity: 1,
      },
    ]);
  };

  // ...
};
```

### Helper Functions

Several helper functions are available in `constants/GLOBAL.js`:

- `getImage(urlPath, fallbackData)` - Handle image URLs with fallbacks
- `getPriceAndCurrency(prices, currencyCode, discount)` - Get price with currency
- `getTitleByLanguage(item, selectedLanguage)` - Get localized title
- `getTranslationString(key, strings, fallbackText)` - Get translation string

Note: Always handle the case when data might be loading or not available yet. Add appropriate checks in your template.

## 🎨 Styling Guidelines

### Theme Colors

Theme colors are set as CSS variables and can be accessed through:

```css
/* Primary color */
bg-primary
text-primary
border-primary

/* Secondary color */
bg-secondary
text-secondary
border-secondary

/* With opacity */
bg-primary/80  /* Primary with 80% opacity */
```

### Responsive Design

Use Tailwind breakpoints for responsive layouts:

- `xxs`: 320px (iPhone SE & small phones)
- `xs`: 375px (iPhone 14 Pro Max)
- `sm`: 640px (Small tablets)
- `tablet`: 744px (iPad Mini portrait)
- `md`: 768px (iPad Mini portrait)
- `lg`: 1024px (iPad Mini landscape)
- `xl`: 1280px (Larger tablets/desktops)

Example of responsive styling:

```jsx
<div
  className="
  grid
  grid-cols-1
  xs:grid-cols-2
  md:grid-cols-3
  lg:grid-cols-4
  gap-4
  p-4
  xs:p-6
  md:p-8
">
  {/* Content */}
</div>
```

### Animation

Use GSAP for animations:

```javascript
import {useEffect, useRef} from "react";
import {useInView} from "react-intersection-observer";
import gsap from "gsap";

const MyComponent = () => {
  const elementRef = useRef(null);
  const {ref, inView} = useInView({
    triggerOnce: true,
    threshold: 0.1,
  });

  useEffect(() => {
    if (!inView) return;

    gsap.from(elementRef.current, {
      y: 20,
      opacity: 0,
      duration: 0.5,
      ease: "power2.out",
    });
  }, [inView]);

  return (
    <div
      ref={(node) => {
        ref(node);
        elementRef.current = node;
      }}>
      {/* Animated content */}
    </div>
  );
};
```

## ⚡ Performance Optimization

### Lazy Loading

Use lazy loading for non-critical components:

```javascript
// At the top of your file
import React, {lazy, Suspense} from "react";

// Lazy load components
const CartModal = lazy(() => import("@/components/shared/modal/CartModal"));

// Simple fallback component
const MinimalLoadingFallback = () => <div className="w-full h-4"></div>;

// In your JSX
{
  showCart && (
    <Suspense fallback={<MinimalLoadingFallback />}>
      <CartModal
        onClose={closeCart}
        cartItems={favorites}
        setCartItems={setFavorites}
      />
    </Suspense>
  );
}
```

### Memoization

Use memoization to prevent unnecessary re-renders:

```javascript
// Memo for components
export default memo(MyComponent);

// useMemo for expensive calculations
const sortedItems = useMemo(() => {
  return [...items].sort((a, b) => a.price - b.price);
}, [items]);

// useCallback for functions passed as props
const handleClick = useCallback(() => {
  // handle click
}, [dependency]);
```

### Conditional Rendering

Only render components when needed:

```jsx
// Conditional rendering
{
  count > 0 && <CartButton count={count} onClick={openModal} />;
}

// Loading states
{
  itemsLoading ? (
    <LoadingComponent />
  ) : itemsData && itemsData.length > 0 ? (
    <FoodGrid items={itemsData} />
  ) : (
    <EmptyState />
  );
}
```

## 🚀 Getting Started

### Prerequisites

- Node.js (v16+)
- npm or yarn

### Installation

1. Clone the repository:

```bash
git clone https://github.com/nullsam/qr.e-menu.ai.git
cd qr.e-menu.ai
```

2. Install dependencies:

```bash
npm install
# or
yarn
```

3. Configure environment variables:

- Create a `.env.local` file based on `.env`
- Set required API endpoints

4. Run the development server:

```bash
npm run dev
# or
yarn dev
```

Open `http://localhost:3000` in your browser to see the documentation. Access a menu via `http://localhost:3000/{slug}`

## 📄 API Requirements

The application expects the following API endpoints:

- `/{slug}/business` - Get business information
- `/{slug}/categories` - Get menu categories
- `/{slug}/items` - Get menu items
- `/{slug}/socials` - Get social media links
- `/{slug}/hours` - Get working hours
- `/{slug}/feedback` - Submit feedback (POST)
- `/{slug}/qr_languages` - Get translations

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

<!-- PDF Generation Script (Hidden in GitHub, visible in editor) -->
<!--
<script src="https://cdnjs.cloudflare.com/ajax/libs/html2pdf.js/0.10.1/html2pdf.bundle.min.js"></script>
<script>
function generatePDF() {
  const element = document.body;
  const opt = {
    margin: [10, 10, 10, 10],
    filename: 'E-Menu_Template_System_Documentation.pdf',
    image: { type: 'jpeg', quality: 0.98 },
    html2canvas: { scale: 2, useCORS: true },
    jsPDF: { unit: 'mm', format: 'a4', orientation: 'portrait' }
  };
  
  // Generate PDF
  html2pdf().from(element).set(opt).save();
}
</script>
-->
