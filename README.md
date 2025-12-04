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

- **artists**: Artist information and metadata
- **videos**: YouTube video details
- **comments**: Raw YouTube comments
- **analyzed_comments**: Comments with sentiment analysis
- **artists_comments_mv**: Materialized view of artist comment aggregations
- **artists_views_mv**: Materialized view of artist view statistics
- **video_comments_mv**: Materialized view of video comment statistics
- **videos_sentiment_mv**: Materialized view of video sentiment analysis

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
