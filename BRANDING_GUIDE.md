# TimeTracker Pro - Branding & Customization Guide

Your TimeTracker Pro application now includes a comprehensive branding system that allows you to customize the appearance and content to match your organization's identity.

## 🎨 Quick Customization

### Step 1: Update Your Branding Configuration

Edit the file `client/src/lib/branding.ts` to customize your application:

```typescript
export const defaultBranding: BrandingConfig = {
  // Company Information
  companyName: "Your Company Name",
  appName: "Your App Name", 
  tagline: "Your Custom Tagline",
  
  // Logo Configuration
  logo: {
    url: "https://yourwebsite.com/logo.png", // Add your logo URL
    text: "Your App Name", // Fallback text if no logo
  },
  
  // Contact Information
  contact: {
    website: "https://yourcompany.com",
    email: "contact@yourcompany.com", 
    phone: "+1 (555) 123-4567",
    address: "123 Business St, City, State 12345",
  },
  
  // Social Media Links
  social: {
    linkedin: "https://linkedin.com/company/yourcompany",
    twitter: "https://twitter.com/yourcompany",
  },
};
```

### Step 2: Choose a Color Theme

The application includes 4 pre-built themes. To change themes, update the `getBranding()` function:

```typescript
export function getBranding(): BrandingConfig {
  // Choose one of these themes:
  return brandingThemes.corporate;     // Blue theme (default)
  return brandingThemes.eco;           // Green theme
  return brandingThemes.professional;  // Purple theme  
  return brandingThemes.tech;          // Orange theme
}
```

## 🎨 Available Themes

### Corporate Theme (Blue)
- **Primary Color**: Blue (`hsl(207, 90%, 54%)`)
- **Best For**: Traditional businesses, corporate environments
- **Feel**: Professional, trustworthy, stable

### Eco Theme (Green) 
- **Primary Color**: Green (`hsl(142, 71%, 45%)`)
- **Best For**: Environmental companies, health organizations
- **Feel**: Natural, growth-oriented, fresh

### Professional Theme (Purple)
- **Primary Color**: Purple (`hsl(262, 83%, 58%)`)
- **Best For**: Creative agencies, tech startups, consulting
- **Feel**: Creative, innovative, premium

### Tech Theme (Orange)
- **Primary Color**: Orange (`hsl(24, 95%, 53%)`)
- **Best For**: Technology companies, creative studios
- **Feel**: Energetic, modern, dynamic

## 🖼️ Adding Your Logo

### Option 1: Logo Image URL
```typescript
logo: {
  url: "https://yourcdn.com/logo.png", // Direct URL to your logo
  text: "Fallback Text", // Shown if image fails to load
}
```

### Option 2: Upload Logo to Project
1. Create an `assets` folder in your project root
2. Add your logo file: `assets/logo.png`
3. Reference it in branding:
```typescript
logo: {
  url: "/assets/logo.png",
  text: "Your Company Name",
}
```

### Option 3: Text-Only Logo
```typescript
logo: {
  text: "Your Brand Name", // Will use text with brand colors
}
```

## 🎨 Custom Color Themes

To create your own color theme, add a new theme to `brandingThemes`:

```typescript
export const brandingThemes = {
  // ... existing themes ...
  
  // Your Custom Theme
  mycompany: {
    ...defaultBranding,
    colors: {
      primary: "hsl(210, 100%, 50%)",        // Your brand color
      primaryForeground: "hsl(0, 0%, 100%)",  // Text on brand color
      secondary: "hsl(210, 20%, 96%)",        // Secondary backgrounds
      accent: "hsl(210, 20%, 96%)",           // Accent elements
      success: "hsl(142, 71%, 45%)",          // Success messages
      warning: "hsl(38, 92%, 50%)",           // Warning messages
      error: "hsl(0, 84%, 60%)",              // Error messages
    },
  },
};
```

Then update `getBranding()` to use your theme:
```typescript
return brandingThemes.mycompany;
```

## 📝 Content Customization

### Landing Page Content
The landing page automatically uses your branding configuration:
- **Hero Title**: Uses `branding.tagline`
- **Company Information**: Uses `branding.companyName`
- **Contact Details**: Uses `branding.contact.*`
- **Features**: Customizable in `branding.features`

### Navigation & Header
- **Logo**: Automatically displays your configured logo
- **App Name**: Uses `branding.appName` or `branding.logo.text`

## 🚀 Advanced Customization

### Custom Fonts
Add custom fonts to your theme:
```typescript
fonts: {
  heading: "'Your Heading Font', system-ui, sans-serif",
  body: "'Your Body Font', system-ui, sans-serif",
}
```

### Dynamic Branding (Multi-Tenant)
For multi-tenant applications, you can load branding dynamically:

```typescript
export function getBranding(): BrandingConfig {
  // Load from environment variables
  const theme = process.env.THEME_NAME || 'corporate';
  return brandingThemes[theme] || brandingThemes.corporate;
  
  // Or load from database/API
  // const branding = await fetchBrandingFromAPI();
  // return branding;
}
```

### Custom CSS Variables
The branding system automatically applies CSS variables. You can use them in custom styles:

```css
.my-custom-element {
  background-color: var(--primary);
  color: var(--primary-foreground);
}
```

## 🌟 Deployment Tips

### Environment-Based Branding
Set different branding for development vs production:

```typescript
export function getBranding(): BrandingConfig {
  if (process.env.NODE_ENV === 'production') {
    return productionBranding;
  }
  return developmentBranding;
}
```

### Brand Asset Optimization
- Use vector logos (SVG) for crisp display at all sizes
- Optimize images for web (WebP format recommended)
- Ensure logos work on both light and dark backgrounds

## 📋 Branding Checklist

- [ ] Update company name and app name
- [ ] Add your logo URL or upload logo file
- [ ] Set contact information (email, phone, website)
- [ ] Choose or create a color theme
- [ ] Customize tagline and description
- [ ] Add social media links
- [ ] Test on different screen sizes
- [ ] Verify logo displays correctly
- [ ] Deploy to production

## 🎯 Brand Consistency Tips

1. **Logo Usage**: Ensure consistent logo sizing across all pages
2. **Color Consistency**: Use theme colors throughout the application
3. **Typography**: Stick to your chosen font family
4. **Contact Information**: Keep contact details up-to-date
5. **Brand Voice**: Maintain consistent messaging and tone

Your TimeTracker Pro application is now fully brandable and ready to represent your organization professionally!