# Tinybird Data Project

A Tinybird data project for real-time analytics using YouTube comments data.

## Overview

This project provides real-time analytics on YouTube comments, tracking sentiment analysis across artists and videos. It includes data sources for artists, videos, comments, and analyzed comments, along with various endpoints and materializations for querying the data.

## Project Structure

```
.
├── datasources/          # Data source definitions (.datasource files)
├── endpoints/           # API endpoints (.pipe files)
├── materializations/    # Materialized views (.pipe files)
├── fixtures/            # Sample data for local testing (.ndjson files)
├── connections/         # External data connections
├── copies/              # Scheduled copy operations
├── sinks/               # Data export configurations
├── pipes/               # Reusable SQL transformations
└── tests/               # Data validation tests
```

## Data Sources

- **artists**: Artist information and metadata (uses ReplacingMergeTree for updates)
- **videos**: YouTube video details (uses ReplacingMergeTree for updates)
- **comments**: Raw YouTube comments
- **analyzed_comments**: Comments with sentiment analysis
- **artists_comments_mv**: Materialized view of artist comment aggregations
- **artists_views_mv**: Materialized view of artist view statistics
- **video_comments_mv**: Materialized view of video comment statistics
- **videos_sentiment_mv**: Materialized view of video sentiment analysis

### About ReplacingMergeTree

This project uses **ReplacingMergeTree** engine for `artists`, `videos`, and `comments` datasources. ReplacingMergeTree is a special ClickHouse table engine that automatically handles updates by keeping only the latest version of rows with the same sorting key.

**Why ReplacingMergeTree?**
- Allows you to update artist follower counts by simply inserting new records with the same `id`
- Enables video view count updates without complex update queries
- Tracks comment like count changes over time
- ClickHouse automatically deduplicates rows during merge operations
- Perfect for dimension tables that receive occasional updates

**How it works:**
Each datasource uses `observed_at` as the version column (configured with `ENGINE_VER "observed_at"`). When you insert a new row with an existing `id`, both rows are temporarily stored. During background merge operations, ClickHouse keeps only the row with the highest `observed_at` timestamp. This ensures you always get the most recent observation.

To force immediate deduplication in queries, use `FINAL`:

```sql
SELECT * FROM artists FINAL WHERE id = '550e8400-e29b-41d4-a716-446655440000'
```

**Important:** Always include a newer `observed_at` timestamp when updating records, otherwise the old record might be kept if it has a more recent timestamp.

Note: Most endpoints automatically handle this, but for direct queries or testing, remember to use `FINAL` for guaranteed latest data.

## Available Endpoints

- **all_artists**: List all artists
- **artists_by_sentiment**: Get artists ranked by sentiment
- **artists_comments**: Get comment statistics by artist
- **artists_views**: Get view statistics by artist
- **unanalyzed_comments**: Retrieve comments pending analysis
- **video_comments**: Get comment statistics by video
- **videos_by_artist**: List videos for a specific artist
- **videos_by_sentiment**: Get videos ranked by sentiment

## Getting Started

### Prerequisites

- Docker or Orbstack (container runtime)
- Linux or macOS
- Tinybird account (create one at [cloud.tinybird.co](https://cloud.tinybird.co))

### Installation

1. Install the Tinybird CLI:
```bash
curl -fsSL https://cli.tinybird.co/install.sh | bash
```

2. Authenticate with your Tinybird account:
```bash
tb login
```

3. Start Tinybird Local for development:
```bash
tb local start
```

### Local Development

1. Build the project locally:
```bash
tb build
```

2. Start the development server:
```bash
tb dev
```

The development server will start at `http://localhost:7181` and automatically reload when you make changes to any datafiles.

3. Test an endpoint locally:
```bash
# Copy your local admin token
tb token copy "admin local_testing@tinybird.co"

# Call an endpoint
curl "http://localhost:7181/v0/pipes/all_artists.json?token=admin%20local_testing@tinybird.co"
```

### Deployment

Deploy to Tinybird Cloud:
```bash
tb --cloud deploy
```

Append data to cloud datasources:
```bash
tb --cloud datasource append <datasource_name> --url <data_url>
# or
tb --cloud datasource append <datasource_name> <file_path>
```

Open your project in Tinybird Cloud UI:
```bash
tb --cloud open
```

## CI/CD

This project includes CI/CD workflows for both GitHub and GitLab:

- **GitHub**: `.github/workflows/tinybird-ci.yml` and `.github/workflows/tinybird-cd.yml`
- **GitLab**: `.gitlab-ci.yml`, `.gitlab/tinybird/tinybird-ci.yml`, and `.gitlab/tinybird/tinybird-cd.yml`

## Usage

### Submitting Data to the API

Tinybird uses the [Events API](https://www.tinybird.co/docs/forward/get-data-in/events-api) to ingest data. You can submit JSON data to any datasource using HTTP POST requests.

#### Submit Artists

```bash
# Local
curl -X POST "http://localhost:7181/v0/events?name=artists&token=admin%20local_testing@tinybird.co" \
  -H "Content-Type: application/json" \
  -d '{
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "name": "Taylor Swift",
    "follower_count": 95000000,
    "observed_at": "2024-01-15T10:30:00"
  }'

# Cloud (replace YOUR_TOKEN with an admin token)
curl -X POST "https://api.tinybird.co/v0/events?name=artists" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "name": "Taylor Swift",
    "follower_count": 95000000,
    "observed_at": "2024-01-15T10:30:00"
  }'
```

#### Submit Videos

```bash
# Local
curl -X POST "http://localhost:7181/v0/events?name=videos&token=admin%20local_testing@tinybird.co" \
  -H "Content-Type: application/json" \
  -d '{
    "id": "660e8400-e29b-41d4-a716-446655440001",
    "name": "Live Performance at MSG",
    "view_count": 1500000,
    "hashtags": ["live", "concert", "music"],
    "music_playing": "Anti-Hero",
    "artist_id": "550e8400-e29b-41d4-a716-446655440000",
    "observed_at": "2024-01-15T10:30:00"
  }'

# Cloud
curl -X POST "https://api.tinybird.co/v0/events?name=videos" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "id": "660e8400-e29b-41d4-a716-446655440001",
    "name": "Live Performance at MSG",
    "view_count": 1500000,
    "hashtags": ["live", "concert", "music"],
    "music_playing": "Anti-Hero",
    "artist_id": "550e8400-e29b-41d4-a716-446655440000",
    "observed_at": "2024-01-15T10:30:00"
  }'
```

#### Submit Comments

```bash
# Local
curl -X POST "http://localhost:7181/v0/events?name=comments&token=admin%20local_testing@tinybird.co" \
  -H "Content-Type: application/json" \
  -d '{
    "id": "770e8400-e29b-41d4-a716-446655440002",
    "name": "music_fan_123",
    "comment_contents": "This performance was absolutely incredible!",
    "likes": 245,
    "video_id": "660e8400-e29b-41d4-a716-446655440001",
    "observed_at": "2024-01-15T10:30:00"
  }'

# Cloud
curl -X POST "https://api.tinybird.co/v0/events?name=comments" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "id": "770e8400-e29b-41d4-a716-446655440002",
    "name": "music_fan_123",
    "comment_contents": "This performance was absolutely incredible!",
    "likes": 245,
    "video_id": "660e8400-e29b-41d4-a716-446655440001",
    "observed_at": "2024-01-15T10:30:00"
  }'
```

#### Updating Artists and Videos (ReplacingMergeTree)

Thanks to ReplacingMergeTree, you can update artist follower counts or video view counts by simply inserting a new record with the same `id`. The engine automatically keeps the latest version based on the `observed_at` timestamp.

```bash
# Update artist follower count - just insert with same id and newer timestamp
curl -X POST "http://localhost:7181/v0/events?name=artists&token=admin%20local_testing@tinybird.co" \
  -H "Content-Type: application/json" \
  -d '{
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "name": "Taylor Swift",
    "follower_count": 96000000,
    "observed_at": "2024-01-15T12:00:00"
  }'

# Update video view count - just insert with same id and newer timestamp
curl -X POST "http://localhost:7181/v0/events?name=videos&token=admin%20local_testing@tinybird.co" \
  -H "Content-Type: application/json" \
  -d '{
    "id": "660e8400-e29b-41d4-a716-446655440001",
    "name": "Live Performance at MSG",
    "view_count": 1550000,
    "hashtags": ["live", "concert", "music"],
    "music_playing": "Anti-Hero",
    "artist_id": "550e8400-e29b-41d4-a716-446655440000",
    "observed_at": "2024-01-15T12:00:00"
  }'
```

The old records will be automatically replaced during ClickHouse's background merge operations. Your queries will always return the latest data based on the most recent `observed_at` timestamp.

### Building a Sentiment Analysis Bot

Create a bot that continuously analyzes unprocessed comments and writes the results back to Tinybird.

#### Example Python Bot

```python
import requests
import time
from datetime import datetime
import uuid

# Configuration
TINYBIRD_BASE_URL = "https://api.tinybird.co"  # or http://localhost:7181 for local
TINYBIRD_TOKEN = "YOUR_TOKEN"  # or "admin local_testing@tinybird.co" for local

def get_unanalyzed_comments():
    """Fetch comments that haven't been analyzed yet."""
    url = f"{TINYBIRD_BASE_URL}/v0/pipes/unanalyzed_comments.json"
    headers = {"Authorization": f"Bearer {TINYBIRD_TOKEN}"}

    response = requests.get(url, headers=headers)
    response.raise_for_status()

    return response.json()["data"]

def analyze_sentiment(comment_text):
    """
    Analyze sentiment of a comment.
    Replace this with your actual sentiment analysis logic
    (e.g., using OpenAI, Hugging Face, or another ML service).
    """
    # This is a placeholder - use your actual sentiment analysis
    # Example: OpenAI, Anthropic Claude, Hugging Face Transformers, etc.

    # Dummy sentiment analysis for demonstration
    if any(word in comment_text.lower() for word in ["amazing", "incredible", "love", "great"]):
        return {
            "sentiment_score": 0.85,
            "sentiment_label": "positive",
            "confidence_score": 0.92
        }
    elif any(word in comment_text.lower() for word in ["bad", "terrible", "hate", "worst"]):
        return {
            "sentiment_score": -0.75,
            "sentiment_label": "negative",
            "confidence_score": 0.88
        }
    else:
        return {
            "sentiment_score": 0.0,
            "sentiment_label": "neutral",
            "confidence_score": 0.65
        }

def submit_analyzed_comment(comment_id, sentiment_data):
    """Submit analyzed comment data to Tinybird."""
    url = f"{TINYBIRD_BASE_URL}/v0/events?name=analyzed_comments"
    headers = {
        "Authorization": f"Bearer {TINYBIRD_TOKEN}",
        "Content-Type": "application/json"
    }

    payload = {
        "comment_id": comment_id,
        "sentiment_score": sentiment_data["sentiment_score"],
        "sentiment_label": sentiment_data["sentiment_label"],
        "confidence_score": sentiment_data["confidence_score"],
        "analyzed_at": datetime.utcnow().isoformat()
    }

    response = requests.post(url, headers=headers, json=payload)
    response.raise_for_status()

    return response.json()

def run_sentiment_bot(interval=10):
    """
    Run the sentiment analysis bot continuously.

    Args:
        interval: Seconds to wait between checks (default: 10)
    """
    print(f"Starting sentiment analysis bot... (checking every {interval}s)")

    while True:
        try:
            # Get unanalyzed comments
            comments = get_unanalyzed_comments()

            if not comments:
                print("No unanalyzed comments found. Waiting...")
                time.sleep(interval)
                continue

            print(f"Processing {len(comments)} comments...")

            # Process each comment
            for comment in comments:
                try:
                    comment_id = comment["id"]
                    comment_text = comment["comment_contents"]

                    print(f"Analyzing comment {comment_id[:8]}...")

                    # Analyze sentiment
                    sentiment = analyze_sentiment(comment_text)

                    # Submit to Tinybird
                    submit_analyzed_comment(comment_id, sentiment)

                    print(f"✓ Comment {comment_id[:8]} analyzed: {sentiment['sentiment_label']}")

                except Exception as e:
                    print(f"✗ Error processing comment {comment_id[:8]}: {e}")
                    continue

            print(f"Batch complete. Waiting {interval}s...")
            time.sleep(interval)

        except KeyboardInterrupt:
            print("\nBot stopped by user.")
            break
        except Exception as e:
            print(f"Error in main loop: {e}")
            time.sleep(interval)

if __name__ == "__main__":
    run_sentiment_bot(interval=10)
```

Save this as `sentiment_bot.py` and run:

```bash
pip install requests
python sentiment_bot.py
```

### Viewing Live Sentiment Data

#### Get Video Sentiment

```bash
# Get sentiment for all videos
curl "http://localhost:7181/v0/pipes/videos_by_sentiment.json?token=admin%20local_testing@tinybird.co"

# Cloud
curl -H "Authorization: Bearer YOUR_TOKEN" \
  "https://api.tinybird.co/v0/pipes/videos_by_sentiment.json"
```

#### Get Artist Sentiment

```bash
# Get sentiment for all artists
curl "http://localhost:7181/v0/pipes/artists_by_sentiment.json?token=admin%20local_testing@tinybird.co"

# Cloud
curl -H "Authorization: Bearer YOUR_TOKEN" \
  "https://api.tinybird.co/v0/pipes/artists_by_sentiment.json"
```

#### Get Detailed Artist Stats

```bash
# Get comment statistics for all artists
curl "http://localhost:7181/v0/pipes/artists_comments.json?token=admin%20local_testing@tinybird.co"

# Get view statistics for all artists
curl "http://localhost:7181/v0/pipes/artists_views.json?token=admin%20local_testing@tinybird.co"
```

#### Get Video Comments

```bash
# Get comment statistics for all videos
curl "http://localhost:7181/v0/pipes/video_comments.json?token=admin%20local_testing@tinybird.co"
```

### Real-time Dashboard Example

You can build a real-time dashboard by polling these endpoints. Here's a simple HTML example:

```html
<!DOCTYPE html>
<html>
<head>
    <title>YouTube Sentiment Dashboard</title>
    <script>
        const TINYBIRD_URL = 'http://localhost:7181';
        const TOKEN = 'admin local_testing@tinybird.co';

        async function fetchArtistSentiment() {
            const response = await fetch(
                `${TINYBIRD_URL}/v0/pipes/artists_by_sentiment.json?token=${encodeURIComponent(TOKEN)}`
            );
            const data = await response.json();
            displayData('artists', data.data);
        }

        async function fetchVideoSentiment() {
            const response = await fetch(
                `${TINYBIRD_URL}/v0/pipes/videos_by_sentiment.json?token=${encodeURIComponent(TOKEN)}`
            );
            const data = await response.json();
            displayData('videos', data.data);
        }

        function displayData(type, data) {
            const container = document.getElementById(type);
            container.innerHTML = `<h2>${type.toUpperCase()}</h2>` +
                data.map(item => `<div>${JSON.stringify(item)}</div>`).join('');
        }

        // Refresh every 5 seconds
        setInterval(() => {
            fetchArtistSentiment();
            fetchVideoSentiment();
        }, 5000);

        // Initial load
        fetchArtistSentiment();
        fetchVideoSentiment();
    </script>
</head>
<body>
    <h1>Real-time Sentiment Dashboard</h1>
    <div id="artists"></div>
    <div id="videos"></div>
</body>
</html>
```

## Testing

Run tests to validate your pipes:
```bash
tb test run
```

## Useful Commands

```bash
# List all datasources
tb datasource ls

# Get endpoint URL
tb endpoint url <pipe_name>

# Get endpoint data
tb endpoint data <pipe_name>

# List all tokens
tb token ls

# Check datasource quarantine
tb datasource data <datasource_name>_quarantine

# Build project
tb build

# Deploy and promote automatically
tb --cloud deploy --wait --auto
```

## Documentation

- [Tinybird Quick Start Guide](https://www.tinybird.co/docs/forward/get-started/quick-start)
- [Core Concepts](https://www.tinybird.co/docs/forward/get-started/concepts)
- [Datafiles Reference](https://www.tinybird.co/docs/forward/dev-reference/datafiles)
- [CLI Commands Reference](https://www.tinybird.co/docs/forward/dev-reference/commands)
- [Tokens & Authentication](https://www.tinybird.co/docs/forward/administration/tokens)

## Environment Variables

Create a `.env.local` file for local development with any secrets:

```bash
# Example secrets
# PRODUCTION_KAFKA_SERVERS=<your-kafka-servers>
# PRODUCTION_KAFKA_USERNAME=<your-username>
# PRODUCTION_KAFKA_PASSWORD=<your-password>
```

## Support

For issues or questions:
- Check the [Tinybird Documentation](https://www.tinybird.co/docs)
- Visit the [Tinybird Community](https://www.tinybird.co/community)
- Contact support at support@tinybird.co

## License

[Your License Here]
