# 🏥 Access Multi-Specialty Medical Clinic Website  
### Version 2.1 — CSS Reorganization & Team Pages Update

## 🎉 What's New in This Update

### ✅ Complete CSS Reorganization
- **Centralized Styling**: All styles moved to `/css/styles.css`
- **Easy Maintenance**: Update styles in one place, applies to all pages
- **Mobile Header Fix**: Header now scrolls away on mobile (no longer sticky)
- **Consistent Design**: All pages now use the same styling

### ✅ Enhanced Navigation
- **"Meet the Team" Dropdown**: Easy access to all provider profiles
- **Three Team Members**:
  - Dr. Michael U. Levinson, M.D. (Psychiatrist)
  - Dr. Ana Nastasia Berezovskaya (Clinical Psychologist)  
  - Oxana Dickey, PMHNP-BC (Psychiatric Nurse Practitioner)

### ✅ New Team Pages
- **team.html**: Beautiful overview page with all team members
- **ana-berezovskaya.html**: Dr. Berezovskaya's professional profile
- **oxana-dickey.html**: Oxana Dickey's professional profile
- **about.html**: Updated Dr. Levinson's page (now uses external CSS)

### ✅ All Pages Updated
Every page now includes:
- External CSS link (no more inline styles!)
- Updated navigation with team dropdown
- Mobile-responsive header
- Consistent branding

---

## 📁 File Structure

```
accessmsmc-website/
│
├── css/
│   └── styles.css              ⭐ NEW - All website styles
│
├── index.html                   ✏️ UPDATED - Uses external CSS
├── treatments.html              ✏️ UPDATED - Uses external CSS  
├── conditions.html              ✏️ UPDATED - Uses external CSS
├── contact.html                 ✏️ UPDATED - Uses external CSS
├── appointment.html             ✏️ UPDATED - Uses external CSS
│
├── about.html                   ✏️ UPDATED - Dr. Levinson profile
├── team.html                    ⭐ NEW - Team overview page
├── ana-berezovskaya.html        ⭐ NEW - Dr. Berezovskaya profile
├── oxana-dickey.html            ⭐ NEW - Oxana Dickey profile
│
├── privacy-policy.html          ⭐ NEW - Privacy Policy (attorney-review placeholder)
├── notice-of-privacy-practices.html ⭐ NEW - HIPAA NPP (attorney-review placeholder)
├── terms.html                   ⭐ NEW - Terms of Use (attorney-review placeholder)
└── accessibility.html           ⭐ NEW - Accessibility statement
```

---

## 🎨 Key Features

### Responsive Design
- ✅ Works on all devices (desktop, tablet, mobile)
- ✅ Mobile header scrolls away (doesn't take up screen space)
- ✅ Touch-friendly navigation
- ✅ Optimized images and layout

### Navigation
- ✅ Dropdown menu for team members
- ✅ Hover effects on desktop
- ✅ Active page highlighting
- ✅ Consistent across all pages

### Team Pages
- ✅ Professional bios and credentials
- ✅ Photo placeholders (ready for real photos)
- ✅ Specialty areas highlighted
- ✅ Consistent design with main site

### Appointment Form
- ✅ All form styles in external CSS
- ✅ Patient intake collected via a HIPAA-BAA-covered Microsoft 365 Form (embedded on `appointment.html`)
- ✅ Professional design
- ✅ Mobile-optimized

---

## 📸 Adding Photos

When you have photos for team members, replace the placeholder divs:

**Find this:**
```html
<div class="photo-placeholder">
    Photo Coming Soon
</div>
```

**Replace with:**
```html
<img src="images/provider-name.jpg" alt="Dr. Name" 
     style="width: 100%; height: auto; border-radius: 16px; 
            box-shadow: 0 10px 40px rgba(0,0,0,0.3);">
```

Recommended photo specs:
- **Size**: 300-400px wide
- **Format**: JPG or WebP
- **Aspect Ratio**: 3:4 (portrait)
- **Quality**: High resolution, professional headshot

---

## 🔧 Customizing Styles

To modify the website appearance, edit `/css/styles.css`:

### Common Customizations

**Change primary color:**
```css
/* Find this gradient */
background: linear-gradient(135deg, #20c997, #17a2b8);

/* Replace with your colors */
background: linear-gradient(135deg, #YOUR_COLOR1, #YOUR_COLOR2);
```

**Modify header:**
```css
/* Find */
header {
    background: linear-gradient(135deg, #1e3c72 0%, #2a5298 100%);
    /* your changes here */
}
```

**Adjust font sizes:**
```css
/* Find specific element classes like */
.doctor-content h1 {
    font-size: clamp(2.5rem, 4vw, 3.2rem);
    /* adjust as needed */
}
```

---

## 🎯 What This Update Solves

### Before ❌
- Inline styles in every page
- Had to update CSS in 9+ places
- Mobile header took up half the screen
- No team member dropdown
- Only one provider page (Dr. Levinson)

### After ✅
- One CSS file controls all pages
- Update styles in one place
- Mobile header scrolls away nicely
- Professional team dropdown navigation
- Three complete provider profiles
- Consistent branding across site

---

## 🔐 Appointment Intake

Patient appointment requests are collected through a Microsoft 365 Form
(covered by Microsoft's HIPAA Business Associate Agreement) embedded via
iframe on `appointment.html`. No PHI is transmitted through this static
site or through any non-BAA third-party vendor. The previous EmailJS +
Google Apps Script pipeline (which sent patient data to a non-BAA email
vendor and a consumer Google Sheet/Calendar) has been retired; see
`REMEDIATION_REPORT.md` for details.

---

## 📱 Mobile Optimization

Special attention to mobile experience:
- Header is no longer sticky on mobile (doesn't block content)
- Touch-friendly navigation
- Responsive grid layouts
- Readable font sizes
- Optimized spacing

---

## 🆘 Troubleshooting

### Styles not loading?
1. Clear browser cache (Ctrl+F5 or Cmd+Shift+R)
2. Verify `/css/styles.css` file exists and uploaded correctly
3. Check file path in HTML: `<link rel="stylesheet" href="css/styles.css">`

### Dropdown not working?
- Dropdown uses CSS `:hover` for desktop
- Should work automatically on hover
- For mobile touch support, JavaScript can be added if needed

### Photos not showing?
- Verify image file paths
- Check image file names match HTML references
- Ensure images are in correct folder (usually `/images/`)

### Appointment form not loading?
- Verify the Microsoft Forms embed URL in `appointment.html` is still valid
- Confirm the Microsoft 365 Form has not been unpublished or moved
- Look for console errors in browser developer tools

---

## 🚀 Future Enhancements

Consider adding:
1. **Real photos** for all team members
2. **Blog section** for mental health articles
3. **Patient testimonials** (with permission)
4. **Insurance providers** page
5. **FAQ section**
6. **Video introductions** from providers

---

## 📞 Clinic Contact

**Access Multi-Specialty Medical Clinic**  
25 Edwards Ct, Suite 101  
Burlingame, CA 94010  
📞 (415) 857-1151  

---

## ✨ Version History

**v2.1** (Current)
- CSS reorganization
- Mobile header fix
- Team member pages added
- Navigation dropdown implemented

**v2.0**
- Appointment system with EmailJS
- Google Sheets/Calendar integration
- Modern design refresh

**v1.0**
- Initial website launch

---

## 📝 Notes

- All pages are HTML/CSS/JavaScript only (no backend required)
- Hosted on GitHub Pages
- Fast loading, SEO-friendly
- Built with WCAG 2.1 AA as a target (keyboard focus states, labeled form fields,
  alt text on images); this has not yet been independently audited — see
  `accessibility.html` for current status
- Works on all modern browsers

---

## 🤝 Need Help?

If you need assistance with:
- Uploading files
- Customizing styles
- Adding photos
- Troubleshooting issues

Just reach out and I'll help you get everything working perfectly!

---

**Last Updated**: January 2026  
**Maintained by**: Access Multi-Specialty Medical Clinic Team


**Access Multi-Specialty Medical Clinic**  
25 Edwards Ct, Suite 101  
Burlingame, CA 94010  
📞 (415) 857-1151  
