from flask import Flask, render_template
import requests
import re
app = Flask(__name__)

def format_text(text):
    """Function for formatting text."""
    if text:
        text = re.sub(r'\s+', ' ', text).strip()
        text = text.replace("revolution", "<b>Revolutions</b>")
    return text

def get_wikipedia_fact():
    try:
        response = requests.get(
            'https://en.wikipedia.org/api/rest_v1/page/random/summary',
            headers={"User-Agent": "Mozilla/5.0"}
        )
        data = response.json()
        return {
            "title": data.get("title"),
            "extract": format_text(data.get("extract"))
        }
    except:
        return {"error": "Failed to fetch data from Wikipedia."}

def get_wikipedia_on_this_day():
    try:
        response = requests.get(
            'https://en.wikipedia.org/api/rest_v1/feed/onthisday/events',
            headers={"User-Agent": "Mozilla/5.0"}
        )
        data = response.json()
        events = data.get("events", [])
        if events:
            random_event = events[0]
            return {
                "year": random_event.get("year"),
                "text": random_event.get("text"),
                "title": random_event["pages"][0]["normalizedtitle"],
                "url": random_event["pages"][0]["content_urls"]["desktop"]["page"]
            }
        return None  
    except:
        return None 
@app.route('/')
def get_fact_and_news():
    fact = get_wikipedia_fact()
    news = get_wikipedia_on_this_day()

    return render_template('index1.html', fact=fact, news=news)

if __name__ == '__main__':
    app.run(debug=True)
