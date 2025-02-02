# Stock Market API Implementation Guide (FastAPI/Flask)

## Core Features

### 1. Market Data Management
- Real-time stock price tracking
- Historical data analysis
- Multiple exchange support (NYSE, NASDAQ, BSE, NSE)
- Company fundamentals
- Volume analysis
- Market indices tracking

### 2. Technical Analysis
- Moving averages (SMA, EMA)
- Relative Strength Index (RSI)
- MACD (Moving Average Convergence Divergence)
- Bollinger Bands
- Support and resistance levels
- Volume-weighted average price (VWAP)

### 3. Portfolio Management
- Portfolio tracking
- Performance analytics
- Risk assessment
- Dividend tracking
- P&L calculations
- Position sizing recommendations

### 4. Trading Signals
- Technical indicator alerts
- Price movement alerts
- Volume spike detection
- Pattern recognition (Head & Shoulders, Double Top/Bottom)
- Momentum indicators
- Volatility analysis

### 5. News and Sentiment Analysis
- Financial news integration
- Social media sentiment analysis
- Market sentiment indicators
- Company news tracking
- Sector-specific news filtering

## Technical Implementation

### 1. Base Project Structure
```
stock_api/
├── app/
│   ├── __init__.py
│   ├── main.py
│   ├── core/
│   │   ├── config.py
│   │   └── security.py
│   ├── models/
│   │   ├── stock.py
│   │   ├── portfolio.py
│   │   └── user.py
│   ├── services/
│   │   ├── market_data.py
│   │   ├── technical_analysis.py
│   │   └── news_service.py
│   └── api/
│       ├── v1/
│       └── endpoints/
├── tests/
└── alembic/
```

### 2. Core Components Implementation

#### Market Data Service
```python
from fastapi import FastAPI, HTTPException
import yfinance as yf
import pandas as pd
from typing import List, Optional

class MarketDataService:
    async def get_real_time_price(self, symbol: str) -> dict:
        try:
            stock = yf.Ticker(symbol)
            data = stock.info
            return {
                "symbol": symbol,
                "current_price": data.get("regularMarketPrice"),
                "change": data.get("regularMarketChange"),
                "volume": data.get("regularMarketVolume"),
                "timestamp": pd.Timestamp.now()
            }
        except Exception as e:
            raise HTTPException(status_code=500, detail=str(e))

    async def get_historical_data(
        self, 
        symbol: str, 
        period: str = "1y",
        interval: str = "1d"
    ) -> List[dict]:
        try:
            stock = yf.Ticker(symbol)
            hist = stock.history(period=period, interval=interval)
            return hist.to_dict(orient="records")
        except Exception as e:
            raise HTTPException(status_code=500, detail=str(e))
```

#### Technical Analysis Service
```python
import pandas as pd
import numpy as np
from typing import Dict, List

class TechnicalAnalysisService:
    def calculate_sma(self, data: pd.DataFrame, window: int) -> pd.Series:
        return data['Close'].rolling(window=window).mean()
    
    def calculate_rsi(self, data: pd.DataFrame, period: int = 14) -> pd.Series:
        delta = data['Close'].diff()
        gain = (delta.where(delta > 0, 0)).rolling(window=period).mean()
        loss = (-delta.where(delta < 0, 0)).rolling(window=period).mean()
        rs = gain / loss
        return 100 - (100 / (1 + rs))
    
    def calculate_macd(
        self, 
        data: pd.DataFrame,
        fast_period: int = 12,
        slow_period: int = 26,
        signal_period: int = 9
    ) -> Dict[str, pd.Series]:
        exp1 = data['Close'].ewm(span=fast_period).mean()
        exp2 = data['Close'].ewm(span=slow_period).mean()
        macd = exp1 - exp2
        signal = macd.ewm(span=signal_period).mean()
        return {
            "macd": macd,
            "signal": signal,
            "histogram": macd - signal
        }
```

#### Portfolio Management Service
```python
from decimal import Decimal
from typing import List, Dict

class PortfolioService:
    async def calculate_portfolio_value(
        self,
        positions: List[Dict],
        market_data_service: MarketDataService
    ) -> Dict:
        total_value = Decimal('0')
        holdings = []
        
        for position in positions:
            current_price = await market_data_service.get_real_time_price(
                position['symbol']
            )
            position_value = (
                Decimal(str(current_price['current_price'])) * 
                Decimal(str(position['quantity']))
            )
            total_value += position_value
            
            holdings.append({
                "symbol": position['symbol'],
                "quantity": position['quantity'],
                "current_value": float(position_value),
                "avg_cost": position['avg_cost'],
                "profit_loss": float(
                    position_value - 
                    (Decimal(str(position['avg_cost'])) * 
                     Decimal(str(position['quantity'])))
                )
            })
        
        return {
            "total_value": float(total_value),
            "holdings": holdings
        }
```

### 3. API Endpoints

```python
from fastapi import APIRouter, Depends, HTTPException
from typing import List, Optional

router = APIRouter()

@router.get("/market-data/{symbol}")
async def get_market_data(
    symbol: str,
    period: Optional[str] = "1d",
    interval: Optional[str] = "1m"
):
    try:
        market_data = await market_data_service.get_real_time_price(symbol)
        return market_data
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))

@router.get("/technical/{symbol}")
async def get_technical_analysis(
    symbol: str,
    indicators: List[str] = Query(["sma", "rsi", "macd"])
):
    try:
        data = await market_data_service.get_historical_data(symbol)
        df = pd.DataFrame(data)
        
        analysis = {}
        if "sma" in indicators:
            analysis["sma"] = technical_service.calculate_sma(df, 20).tolist()
        if "rsi" in indicators:
            analysis["rsi"] = technical_service.calculate_rsi(df).tolist()
        if "macd" in indicators:
            analysis["macd"] = technical_service.calculate_macd(df)
            
        return analysis
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))

@router.get("/portfolio/{user_id}")
async def get_portfolio(
    user_id: int,
    current_user: User = Depends(get_current_user)
):
    if current_user.id != user_id:
        raise HTTPException(status_code=403, detail="Not authorized")
    
    try:
        positions = await portfolio_service.get_user_positions(user_id)
        portfolio_value = await portfolio_service.calculate_portfolio_value(
            positions,
            market_data_service
        )
        return portfolio_value
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))
```

### 4. Advanced Features

#### Pattern Recognition
```python
class PatternRecognitionService:
    def identify_head_and_shoulders(self, data: pd.DataFrame) -> List[Dict]:
        # Implementation of head and shoulders pattern recognition
        pass
    
    def identify_double_top(self, data: pd.DataFrame) -> List[Dict]:
        # Implementation of double top pattern recognition
        pass
    
    def identify_support_resistance(self, data: pd.DataFrame) -> Dict:
        # Implementation of support and resistance levels
        pass
```

#### News Sentiment Analysis
```python
from textblob import TextBlob
import newsapi

class NewsSentimentService:
    def __init__(self, news_api_key: str):
        self.newsapi = newsapi.NewsApiClient(api_key=news_api_key)
    
    async def get_company_sentiment(self, symbol: str) -> Dict:
        news = self.newsapi.get_everything(
            q=symbol,
            language='en',
            sort_by='publishedAt'
        )
        
        sentiments = []
        for article in news['articles']:
            analysis = TextBlob(article['title'] + " " + article['description'])
            sentiments.append(analysis.sentiment.polarity)
        
        return {
            "average_sentiment": sum(sentiments) / len(sentiments),
            "sentiment_count": len(sentiments),
            "latest_articles": news['articles'][:5]
        }
```

### 5. Real-time Updates with WebSockets

```python
from fastapi import FastAPI, WebSocket
from typing import Dict
import json

class StockUpdateManager:
    def __init__(self):
        self.active_connections: Dict[str, List[WebSocket]] = {}
    
    async def connect(self, websocket: WebSocket, symbol: str):
        await websocket.accept()
        if symbol not in self.active_connections:
            self.active_connections[symbol] = []
        self.active_connections[symbol].append(websocket)
    
    async def disconnect(self, websocket: WebSocket, symbol: str):
        self.active_connections[symbol].remove(websocket)
    
    async def broadcast_update(self, symbol: str, data: dict):
        if symbol in self.active_connections:
            for connection in self.active_connections[symbol]:
                await connection.send_json(data)

@app.websocket("/ws/{symbol}")
async def websocket_endpoint(websocket: WebSocket, symbol: str):
    await manager.connect(websocket, symbol)
    try:
        while True:
            data = await market_data_service.get_real_time_price(symbol)
            await manager.broadcast_update(symbol, data)
            await asyncio.sleep(1)
    except WebSocketDisconnect:
        await manager.disconnect(websocket, symbol)
```

## Setup Instructions

1. Create virtual environment and install dependencies:
```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install fastapi uvicorn sqlalchemy databases yfinance pandas numpy textblob newsapi-python
```

2. Set up environment variables:
```bash
export STOCK_API_KEY=your_api_key
export NEWS_API_KEY=your_news_api_key
export DATABASE_URL=postgresql://user:password@localhost/stockdb
```

3. Initialize the database:
```bash
alembic upgrade head
```

4. Run the application:
```bash
uvicorn app.main:app --reload
```

## Testing Strategy

1. Unit Tests:
```python
def test_technical_analysis():
    service = TechnicalAnalysisService()
    data = pd.DataFrame({
        'Close': [100, 102, 98, 103, 99, 105]
    })
    
    sma = service.calculate_sma(data, 3)
    assert len(sma) == len(data)
    assert not pd.isna(sma.iloc[-1])
    
    rsi = service.calculate_rsi(data)
    assert all(0 <= x <= 100 for x in rsi.dropna())
```

2. Integration Tests:
```python
async def test_portfolio_value():
    market_data = MarketDataService()
    portfolio = PortfolioService()
    
    positions = [
        {"symbol": "AAPL", "quantity": 10, "avg_cost": 150.0},
        {"symbol": "GOOGL", "quantity": 5, "avg_cost": 2500.0}
    ]
    
    result = await portfolio.calculate_portfolio_value(positions, market_data)
    assert "total_value" in result
    assert "holdings" in result
    assert len(result["holdings"]) == len(positions)
```

This implementation provides a robust foundation for a stock market API with real-time data, technical analysis, portfolio management, and sentiment analysis capabilities. The modular structure allows for easy expansion and maintenance.
