# 🏥 Access Multi-Specialty Medical Clinic Website  
### Version 2 — Modernized, automated appointment workflow

This repository contains the full source code for the **Access Multi-Specialty Medical Clinic** public website, built as a lightweight, fast, GitHub Pages–hosted site with enhanced patient appointment handling and automated workflows.

The website is designed to provide:  
- Clear and accessible information about treatments & conditions  
- Professional online presence for Dr. Levinson’s clinic  
- A comprehensive **Appointment Request System** with automated:  
  - Clinic notifications  
  - Patient confirmations  
  - Google Sheets data logging  
  - Google Calendar event creation  

---

## 🚀 Live Website  
**GitHub Pages Deployment:**  
🔗 https://accessmsmc.github.io/accessmsmc-website/

This branch (`version2`) reflects the upgraded site with improved navigation, redesigned appointment form, and background automations.

---

## 🧩 Key Features

### ✔️ Beautiful, Mobile-Responsive Design  
- Custom header and navigation across all pages  
- Clean typography and layout  
- Optimized for speed (static HTML/CSS/JS)

### ✔️ Dynamic Appointment Request System  
Built using **vanilla JavaScript**, the form includes:

- Full validation (email, phone, required fields, etc.)
- Detailed patient intake fields (conditions, medications, treatments)
- Smooth UI feedback for success and errors
- Automatic form reset after submission

### ✔️ Email Automation with EmailJS  
The site uses **EmailJS** (no backend required) to send:

1. **Clinic Notification Email**  
   - Detailed patient request  
   - Triage information  
   - Clinical history  
   - Action required within 24 hours  

2. **Patient Confirmation Email**  
   - Appointment request summary  
   - Clinic contact instructions  
   - Location & next steps  
   - HIPAA-compliant disclaimer  

All using pre-designed, branded HTML templates.

### ✔️ Google Sheets Logging  
Every appointment request is automatically logged:

- Submission date  
- Patient name  
- Contact info  
- Insurance details  
- Clinical notes  
- Preferred appointment date  
- Status column (`New`, `Called`, `Confirmed`, etc.)

### ✔️ Google Calendar Event Automation  
Using a Google Apps Script webhook, each form submission automatically creates:

- A calendar entry on the preferred appointment date  
- Patient’s name and appointment type  
- Included clinical and contact details  

---

## 🗂️ Repository Structure

```
accessmsmc-website/
│
├── index.html                
├── about.html                
├── treatments.html           
├── conditions.html           
├── contact.html              
├── appointment.html          
│
├── styles/                   
├── images/                   
│
├── config.js                 
└── README.md                 
```

---

## ⚙️ Technologies Used

### Frontend
- HTML5  
- CSS3  
- JavaScript (ES6)  

### Automation / Backend-like Services
- EmailJS  
- Google Apps Script  
- Google Sheets API  
- Google Calendar API  

### Hosting  
- GitHub Pages  

---

## 🔐 Security Considerations

- Uses EmailJS **Public Keys only**  
- Google Apps Script endpoint locked down  
- No PHI stored on the website itself  

---

## 📦 Development Workflow

1. Clone:

   ```bash
   git clone https://github.com/accessmsmc/accessmsmc-website.git
   cd accessmsmc-website
   ```

2. Modify HTML/CSS/JS  
3. Commit + push:

   ```bash
   git add .
   git commit -m "Update"
   git push origin version2
   ```

4. GitHub Pages redeploys automatically.

---

## 🤝 Contributions

Pull requests and issues are welcome.

---

## 🩺 Clinic Contact

**Access Multi-Specialty Medical Clinic**  
25 Edwards Ct, Suite 101  
Burlingame, CA 94010  
📞 (415) 857-1151  
