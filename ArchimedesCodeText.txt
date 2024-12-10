import pandas as pd
from typing import List, Dict, Tuple
 
class SentimentStockAnalyzer:
    def __init__(self):
        # Dictionary of sentiment words and their scores (1-10)
        self.sentiment_dict = {
            # Negative sentiment (1-4)
            'crash': 1, 'plunge': 1, 'bankruptcy': 1, 'crisis': 1,
            'decline': 2, 'weak': 2, 'downturn': 2, 'bearish': 2,
            'decrease': 3, 'cautious': 3, 'concern': 3, 'volatile': 3,
            'uncertain': 4, 'mixed': 4, 'challenging': 4, 'slow': 4,
            # Neutral sentiment (5-6)
            'stable': 5, 'steady': 5, 'maintain': 5, 'unchanged': 5,
            'potential': 6, 'opportunity': 6, 'developing': 6, 'emerging': 6,
            # Positive sentiment (7-10)
            'improve': 7, 'gain': 7, 'positive': 7, 'upgrade': 7,
            'growth': 8, 'bullish': 8, 'outperform': 8, 'beat': 8,
            'surge': 9, 'soar': 9, 'breakthrough': 9, 'exceptional': 9,
            'record': 10, 'revolutionary': 10, 'outstanding': 10, 'stellar': 10
        }
    def analyze_headline(self, headline: str) -> float:
        """
        Analyze a single headline and return its sentiment score.
        """
        words = headline.lower().split()
        scores = []
        for word in words:
            if word in self.sentiment_dict:
                scores.append(self.sentiment_dict[word])
        return sum(scores) / len(scores) if scores else 5.0  # Default to neutral if no sentiment words found
    def analyze_headlines(self, headlines: List[str]) -> float:
        """
        Analyze multiple headlines and return average sentiment score.
        """
        scores = [self.analyze_headline(headline) for headline in headlines]
        return sum(scores) / len(scores) if scores else 5.0
    def analyze_stock_metrics(self, current_price: float, avg_price_50d: float, 
                            volume: float, avg_volume_50d: float) -> float:
        """
        Analyze stock metrics and return a technical score (1-10).
        """
        # Price trend score (1-10)
        price_ratio = current_price / avg_price_50d
        price_score = min(10, max(1, price_ratio * 5))
        # Volume trend score (1-10)
        volume_ratio = volume / avg_volume_50d
        volume_score = min(10, max(1, volume_ratio * 5))
        # Combined technical score
        return (price_score * 0.6 + volume_score * 0.4)
    def get_recommendation(self, headlines: List[str], current_price: float, 
                          avg_price_50d: float, volume: float, 
                          avg_volume_50d: float) -> Tuple[str, Dict]:
        """
        Generate buy/hold/sell recommendation based on sentiment and technical analysis.
        """
        # Get scores
        sentiment_score = self.analyze_headlines(headlines)
        technical_score = self.analyze_stock_metrics(current_price, avg_price_50d, 
                                                   volume, avg_volume_50d)
        # Combined score (60% sentiment, 40% technical)
        final_score = sentiment_score * 0.6 + technical_score * 0.4
        # Generate recommendation
        if final_score >= 7:
            recommendation = "BUY"
        elif final_score <= 4:
            recommendation = "SELL"
        else:
            recommendation = "HOLD"
        # Return recommendation and analysis details
        return recommendation, {
            'sentiment_score': round(sentiment_score, 2),
            'technical_score': round(technical_score, 2),
            'final_score': round(final_score, 2),
            'price_vs_50d': round((current_price/avg_price_50d - 1) * 100, 2),
            'volume_vs_50d': round((volume/avg_volume_50d - 1) * 100, 2)
        }
 
# Example usage
if __name__ == "__main__":
    analyzer = SentimentStockAnalyzer()
    # Example data
    headlines = [
        "Company XYZ reports stellar quarterly earnings",
        "New breakthrough product launched by XYZ",
        "Market analysts cautious about sector outlook"
    ]
    current_price = 150
    avg_price_50d = 140
    volume = 1000000
    avg_volume_50d = 800000
    recommendation, details = analyzer.get_recommendation(
        headlines, current_price, avg_price_50d, volume, avg_volume_50d
    )
    print(f"Recommendation: {recommendation}")
    print("Analysis Details:")
    for key, value in details.items():
        print(f"{key}: {value}")
