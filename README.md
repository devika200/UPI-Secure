# 🔐 UPI Fraud Detection System

AI-powered real-time fraud detection for UPI transactions using Hidden Markov Models and behavioral analysis.

## 🌟 Features

- **Real-time Fraud Detection** - Analyze transactions instantly
- **User Authentication** - Secure JWT-based login system
- **Transaction History** - Track all your transactions
- **ML-Powered Analysis** - Uses Hidden Markov Models for pattern detection
- **Behavioral Learning** - Learns from your transaction patterns
- **Risk Scoring** - Provides detailed risk assessment

## 🛠️ Tech Stack

### Frontend
- React 18
- Vite
- Axios
- React Router

### Backend
- Flask (Python)
- Flask-JWT-Extended
- Flask-CORS
- MongoDB (PyMongo)

### Machine Learning
- Hidden Markov Models (HMM)
- Conditional Random Fields (CRF)
- scikit-learn
- pandas, numpy
- hmmlearn
- sklearn-crfsuite

## 📦 Project Structure

```
UPI-Secure/
├── Backend/
│   ├── app/
│   │   ├── __init__.py         # Flask application factory
│   │   ├── config.py           # Configuration management
│   │   ├── database.py         # MongoDB connection manager
│   │   ├── models.py           # ML model loader (ModelManager)
│   │   ├── utils.py            # Feature calculation utilities
│   │   ├── routes/
│   │   │   ├── auth.py         # Authentication endpoints
│   │   │   ├── fraud.py        # Fraud detection endpoints
│   │   │   └── admin.py        # Admin endpoints
│   │   └── services/
│   │       ├── fraud_detection.py  # FraudDetectionService
│   │       └── risk_analysis.py    # RiskAnalysisService
│   ├── models/
│   │   ├── arlg_hmm_model.pkl  # Trained AR-HMM model
│   │   ├── crf_model.pkl       # Trained CRF model
│   │   ├── scaler.pkl          # Feature scaler
│   │   └── label_encoder.pkl   # Label encoder
│   ├── mainapp.py              # Entry point (imports from wsgi)
│   ├── wsgi.py                 # WSGI application
│   ├── model.py                # AutoRegressiveHMM class
│   └── requirements.txt        # Python dependencies
├── frontend/
│   ├── src/
│   │   ├── pages/              # React pages (Home, Login, Register, TransactionForm, History, About)
│   │   ├── components/         # React components (Navbar)
│   │   ├── App.jsx             # Main app component
│   │   └── config.js           # API configuration
│   ├── package.json
│   └── vite.config.js
├── models/                     # Additional model copies
└── UPI_SECURE.ipynb            # Model training notebook
```

## 🚀 Quick Start

### Prerequisites
- Python 3.12+ (tested with 3.14)
- Node.js 18+
- MongoDB (local or Atlas)

### Backend Setup
```bash
cd Backend
python -m venv venv
venv\Scripts\activate  # Windows
pip install -r requirements.txt
python mainapp.py
```

Backend runs on: `http://localhost:5000`

### Frontend Setup
```bash
cd frontend
npm install
npm run dev
```

Frontend runs on: `http://localhost:5173`

## 🌐 Deployment

See [DEPLOYMENT_GUIDE.md](DEPLOYMENT_GUIDE.md) for complete deployment instructions.

**Quick Deploy:**
- Frontend → Netlify
- Backend → Render
- Database → MongoDB Atlas

All free tier available!

## 📊 How It Works

1. **User Registration/Login** - Secure authentication with JWT
2. **Transaction Submission** - User submits transaction details
3. **Feature Extraction** - System calculates 10 features from transaction
4. **Historical Analysis** - Retrieves last 2 transactions for lagging
5. **Ensemble ML Prediction** - HMM and CRF models analyze patterns
6. **Risk Assessment** - Returns fraud score, risk factors, and confidence level
7. **History Tracking** - Saves transaction with prediction for future learning

## 🔒 Security Features

- Bcrypt password hashing
- JWT token authentication
- CORS protection
- Input validation
- SQL injection prevention
- XSS protection

## 📈 ML Model Details

**Algorithms**: Ensemble of HMM (Hidden Markov Model) and CRF (Conditional Random Field)

**Features Used** (10 total):
1. **Transaction Amount (INR)** - Current transaction amount
2. **Transaction_Amount_Diff** - Absolute difference from user's last transaction
3. **Transaction_Frequency_Score** - Recent transactions in 30 days / 10
4. **Time_Anomaly_Score** - Unusual hour detection scaled by amount ratio
5. **Recipient_Total_Transactions** - Count of transactions to this recipient
6. **Recipient_Avg_Transaction_Amount** - Average amount to this recipient
7. **Risk_Score** - (Frequency_Score + Time_Anomaly_Score) / 2
8. **hour** - Hour of day (0-23)
9. **day_of_week** - Day of week (0-6, Monday=0)
10. **Location_Cluster** - Placeholder (currently 0.0)

**Classification Labels**:
- 0: Normal (Low fraud risk)
- 1: Suspicious (Medium fraud risk)
- 2: Fraud (High fraud risk)

**Ensemble Scoring**:
- **HMM prediction**: Uses AR-HMM with 3 lags (requires ≥3 historical transactions)
  - Retrieves last 2 transactions from MongoDB
  - Creates lagged feature matrix: [t-2, t-1, t] features
  - Applies scaler to all rows
  - Creates 30-feature lagged observation: np.hstack([X[-1], X[-2], X[-3]])
  - Predicts state (0, 1, or 2)
- **CRF prediction**: Uses current transaction's 10 scaled features
  - Converts to dictionary format with exact attribute names
  - Predicts label (0, 1, or 2)
- **Final score**: Average of HMM and CRF probabilities (0.0-1.0)
  - HMM probability = state / 2.0
  - CRF probability = label / 2.0
  - Ensemble score = (HMM_prob + CRF_prob) / 2
- **Classification**:
  - Score ≥0.67 → Label 2 (Fraud), High confidence
  - Score 0.33-0.67 → Label 1 (Suspicious), Medium confidence
  - Score <0.33 → Label 0 (Normal), Low confidence

## 🎯 API Endpoints

### Authentication
- `POST /api/register` - Register new user
- `POST /api/login` - User login

### Fraud Detection
- `POST /api/check_fraud` - Check transaction for fraud (requires JWT)
- `GET /api/history` - Get user's transaction history (requires JWT)

### Admin
- `GET /health` - Health check (model status, DB connection)
- `GET /api/admin/stats` - Database statistics (users, transactions, fraud rate)
- `GET /api/admin/users` - Get all users (without passwords)
- `GET /api/admin/transactions` - Get all transactions with filters

### Testing
- `POST /api/fraud/predict` - Public fraud prediction endpoint (no auth required)

## 🧪 Testing

**Test the API** using the public endpoint:
```bash
curl -X POST http://localhost:5000/api/fraud/predict \
  -H "Content-Type: application/json" \
  -d '{
    "username": "test_user",
    "recipient_id": "merchant@paytm",
    "transaction_amount": 5000,
    "transaction_time": "2024-01-15T14:30:00"
  }'
```

**Check health status**:
```bash
curl http://localhost:5000/health
```

## 📝 Environment Variables

### Backend (.env)
```bash
MONGODB_URI=mongodb://localhost:27017/
JWT_SECRET_KEY=your-secret-key-change-in-production
SECRET_KEY=dev-secret-key-change-in-production
FLASK_ENV=development
PORT=5000
```

### Frontend (src/config.js)
```javascript
export const API_URL = 'http://localhost:5000';
```

## 🤝 Contributing

1. Fork the repository
2. Create feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit changes (`git commit -m 'Add AmazingFeature'`)
4. Push to branch (`git push origin feature/AmazingFeature`)
5. Open Pull Request

## 📄 License

This project is licensed under the MIT License.

## 👥 Authors

- Your Name - Initial work

## 🙏 Acknowledgments

- Hidden Markov Model implementation using hmmlearn
- Flask framework for backend API
- React for frontend UI
- MongoDB for data storage

## 📞 Support

For issues and questions:
- Open an issue on GitHub
- Check [DEPLOYMENT_GUIDE.md](DEPLOYMENT_GUIDE.md)
- Review [GITHUB_CHECKLIST.md](GITHUB_CHECKLIST.md)

---

Made with ❤️ for secure UPI transactions
