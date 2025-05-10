import requests
import argparse
import time
import json
import os


NEWS_API_KEY = os.environ.get("NEWS_API_KEY", "#コード")
NEWS_API_ENDPOINT = "https://newsapi.org/v2/everything"



def fetch_news(keywords, num_articles=5):
    """指定されたキーワードでNewsAPIから記事を取得する"""
    params = {
        'q': keywords,
        'sortBy': 'relevancy',
        'pageSize': num_articles,
        'language': 'ja',
        'apiKey': NEWS_API_KEY
    }
    try:
        response = requests.get(NEWS_API_ENDPOINT, params=params)
        response.raise_for_status()
        data = response.json()

        if data['status'] == 'ok':
            return data['articles']
        else:
            print(f"API Error: {data.get('message', 'Unknown error')}")
            return None
    except requests.exceptions.RequestException as e:
        print(f"Network Error: {e}")
        return None
    except json.JSONDecodeError:
        print("Error decoding API response.")
        return None

def simple_summarize(text, num_sentences=2):
    if not text:
        return "No content available for summary."

    sentences = text.replace('\n', '.').split('.')
    sentences = [s.strip() for s in sentences if s.strip()]

    if not sentences:
        return "Could not extract sentences for summary."

    summary = ". ".join(sentences[:num_sentences]) + "."
    return summary

def display_articles(articles):
    if not articles:
        print("No articles found.")
        return

    print("\n--- Relevant News Articles ---")
    for i, article in enumerate(articles):
        title = article.get('title', 'No Title')
        url = article.get('url', 'No URL')
        content_to_summarize = article.get('description') or article.get('content', '')

        print(f"\n{i+1}. {title}")
        print(f"   URL: {url}")
        summary = simple_summarize(content_to_summarize)
        print(f"   Summary: {summary}")
    print("----------------------------\n")

def run_pomodoro(work_minutes=25, break_minutes=5):
    try:
        print(f"Starting Pomodoro: {work_minutes} minutes work, {break_minutes} minutes break.")
        print(f"Work session started. Focus!")
        time.sleep(work_minutes * 60)
        print("\nWork session finished! Time for a break.")
        time.sleep(break_minutes * 60)
        print("Break finished! Ready for the next session?")
    except KeyboardInterrupt:
        print("\nPomodoro timer interrupted.")


def main():
    parser = argparse.ArgumentParser(description="Smart Study Session Helper - Fetches news and runs a Pomodoro timer.")
    parser.add_argument("keywords", type=str, help="Keywords to search for news articles.")
    parser.add_argument("-n", "--num-articles", type=int, default=3, help="Number of articles to fetch (default: 3).")
    parser.add_argument("-p", "--pomodoro", action="store_true", help="Start a Pomodoro timer after fetching news.")
    parser.add_argument("-w", "--work-minutes", type=int, default=25, help="Work duration for Pomodoro (minutes, default: 25).")
    parser.add_argument("-b", "--break-minutes", type=int, default=5, help="Break duration for Pomodoro (minutes, default: 5).")

    args = parser.parse_args()

    if NEWS_API_KEY == "YOUR_NEWS_API_KEY_HERE":
        print("Error: Please set your NewsAPI key in the script or as an environment variable.")
        return

    articles = fetch_news(args.keywords, args.num_articles)

    if articles:
        display_articles(articles)

        if args.pomodoro:
            run_pomodoro(args.work_minutes, args.break_minutes)
    else:
        print(f"Could not fetch articles for keywords: {args.keywords}")

if __name__ == "__main__":
    main()
