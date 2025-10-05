# 🎨 Logo Integration Instructions

## 📥 Save the Logo Images

Please save the two images you provided to these exact locations:

### 1. ARESx Cyber Team Logo (Red Dragon Shield)
**Save as**: `d:\Portfolio\colonneil\assets\images\logos\aresx-logo.png`
- This is the red dragon shield logo
- Recommended size: 400x400px or similar square dimensions

### 2. Cytailor.io Logo (Cyan Circular Design)  
**Save as**: `d:\Portfolio\colonneil\assets\images\logos\cytailor-logo.png`
- This is the cyan/teal colored circular logo with "CyTailor" text
- Recommended size: 400x400px or similar square dimensions

---

## 🖼️ How to Save:

### Option 1: Drag & Drop
1. Open File Explorer and navigate to: `d:\Portfolio\colonneil\assets\images\logos\`
2. Drag the images from your downloads/desktop to this folder
3. Rename them to exactly:
   - `aresx-logo.png`
   - `cytailor-logo.png`

### Option 2: PowerShell Command
If you have the images ready, you can use:

```powershell
# Copy ARESx logo
Copy-Item "PATH_TO_YOUR_ARESX_IMAGE" -Destination "d:\Portfolio\colonneil\assets\images\logos\aresx-logo.png"

# Copy Cytailor logo
Copy-Item "PATH_TO_YOUR_CYTAILOR_IMAGE" -Destination "d:\Portfolio\colonneil\assets\images\logos\cytailor-logo.png"
```

---

## ✅ What I've Already Updated:

### About Page (`_pages/about.md`)
- ✨ Added logo banner section at the top showing both logos
- 🎨 Styled with background and hover effects
- 🔗 Made logos clickable linking to respective websites
- 📱 Responsive design that works on all screen sizes

### HackTheBox Integration
- 🏆 Added live HackTheBox badge that shows your current rank
- 🔗 Badge links directly to your HTB profile

---

## 🔍 Preview Structure:

```
About Page Layout:
├── Header Image (Utah photo)
├── 🎨 LOGO BANNER
│   ├── Cytailor.io Logo (clickable)
│   └── ARESx Logo (clickable)
├── Introduction with roles
├── 🏆 CTF Section
│   ├── HackTheBox Badge (live)
│   ├── Platform achievements
│   └── Team info
├── 💼 Professional Experience
└── ...rest of content
```

---

## 🚀 After Saving the Images:

1. **Test locally**:
   ```powershell
   cd d:\Portfolio\colonneil
   bundle exec jekyll serve
   ```
   Visit: http://127.0.0.1:4000/about/

2. **Deploy to live site**:
   ```powershell
   git add assets/images/logos/
   git commit -m "Add Cytailor and ARESx logos to About page"
   git push origin master
   ```

Your logos will appear on https://colonneil.me/about/ within 1-2 minutes! 🎉

---

## 📊 Logo Display Specifications:

- **Height**: 80px (auto-width to maintain aspect ratio)
- **Spacing**: 20px margin on each side
- **Background**: Semi-transparent dark overlay for contrast
- **Border Radius**: 10px rounded corners
- **Hover Effect**: Slight opacity change on mouse hover
- **Links**: Opens in new tab when clicked

---

Need help with anything else? Just let me know! 🚀
