# 🏋️ FitBuddy – AI Fitness Planner

FitBuddy is an AI-powered fitness planning web application that generates personalized workout plans based on a user's age, fitness goal, fitness level, and number of training days per week.

The application uses **FastAPI** for the backend, **Google Gemini AI** for fitness-plan generation, and **HTML/CSS/JavaScript** for the frontend.

---

## 🚀 Features

- 🤖 AI-generated personalized fitness plans
- 🎯 Multiple fitness goals such as:
  - Hypertrophy
  - Weight Loss
  - Strength
  - General Fitness
- 🏋️ Fitness-level selection
  - Beginner
  - Intermediate
  - Advanced
- 📅 Custom number of workout days per week
- 👤 Age-based workout planning
- ⚡ FastAPI REST API
- 🌐 Web-based frontend
- 🎨 Responsive fitness dashboard
- 🔐 Environment-variable based API key configuration
- 🗄️ SQLAlchemy async database support

---

## 🛠️ Technologies Used

### Backend

- Python
- FastAPI
- Uvicorn
- SQLAlchemy
- Pydantic
- Google Gemini API

### Frontend

- HTML5
- CSS3
- JavaScript

### Database

- SQLAlchemy AsyncSession
- Database URL configured through `.env`

---

## 📁 Project Structure

```text
Fitbuddy/
│
├── app/
│   ├── __init__.py
│   ├── main.py
│   │
│   ├── api/
│   │   ├── __init__.py
│   │   └── plans.py
│   │
│   ├── core/
│   │   ├── __init__.py
│   │   └── config.py
│   │
│   ├── db/
│   │   ├── __init__.py
│   │   └── session.py
│   │
│   └── services/
│       ├── __init__.py
│       └── gemini_service.py
│
├── static/
│   ├── js/
│   │   └── script.js
│   │
│   └── style.css
│
├── templates/
│   └── index.html
│
├── .env
├── .gitignore
├── requirements.txt
└── README.md
