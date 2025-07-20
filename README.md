# Soccer Data Scraper

A comprehensive Scrapy-based web scraper for extracting soccer match data from [stat-football.com](http://stat-football.com/en/). This project features 62 leagues globally and provides both data cleaning and database storage capabilities.

## 📊 Dataset

The scraped dataset is publicly available on ScrapingHub: [https://app.scrapinghub.com/datasets/RS3NYdirkoz](https://app.scrapinghub.com/datasets/RS3NYdirkoz)

## 🏗️ Project Structure

```
scrapy-soccer/
├── soccer/                    # Main Scrapy project
│   ├── __init__.py
│   ├── items.py              # Data models
│   ├── middlewares.py        # Custom middlewares
│   ├── pipelines.py          # Data processing pipelines
│   ├── run.py                # Entry point for running the spider
│   ├── settings.py           # Scrapy configuration
│   └── spiders/
│       ├── __init__.py
│       └── MatchSpider.py    # Main spider implementation
├── scrapy.cfg                # Scrapy deployment configuration
├── requirements.txt          # Python dependencies
├── docs/                     # Documentation
├── diagrams/                 # Mermaid diagrams
└── README.md                 # This file
```

## 🚀 Quick Start

### Prerequisites

- Python 3.7+
- pip

### Installation

1. Clone the repository:
```bash
git clone https://github.com/zseta/scrapy-soccer.git
cd scrapy-soccer
```

2. Install dependencies:
```bash
pip install -r requirements.txt
```

3. Run the spider:
```bash
python soccer/run.py
```

Or using Scrapy directly:
```bash
scrapy crawl soccer
```

## 📋 Features

### Data Extraction
- **62 Global Leagues**: Comprehensive coverage of soccer leagues worldwide
- **Match Details**: Date, teams, goals (first half and full time)
- **Country & League Information**: Organized data structure
- **Robust Parsing**: Handles various match states (postponed, awarded, annulled)

### Data Processing
- **Cleaning Pipeline**: Automatic data cleaning and validation
- **Database Pipeline**: MySQL storage capability (optional)
- **HTTP Caching**: 24-hour cache to reduce server load
- **Error Handling**: Graceful handling of invalid data

### Technical Features
- **Robots.txt Compliance**: Respects website crawling policies
- **User-Agent Identification**: Proper bot identification
- **Modular Architecture**: Clean separation of concerns
- **Configurable Settings**: Easy customization via settings.py

## 📊 Data Model

The scraper extracts the following data for each match:

| Field | Description | Example |
|-------|-------------|---------|
| `country` | Country of the league | "England" |
| `league` | League name | "Premier League" |
| `date` | Match date (YYYY-MM-DD) | "2023-12-25" |
| `home_team` | Home team name | "Manchester United" |
| `away_team` | Away team name | "Liverpool" |
| `home_goals1` | Home team first half goals | 1 |
| `home_goals2` | Home team second half goals | 1 |
| `away_goals1` | Away team first half goals | 0 |
| `away_goals2` | Away team second half goals | 1 |

## ⚙️ Configuration

### Basic Settings
Edit `soccer/settings.py` to customize:

```python
# Database configuration (optional)
DB_SETTINGS = {
    'db': 'soccer_db',
    'user': 'soccer_user',
    'passwd': 'soccer_pass',
    'host': '0.0.0.0',
}

# Enable database pipeline
ITEM_PIPELINES = {
    'soccer.pipelines.CleaningPipeline': 300,
    'soccer.pipelines.DatabasePipeline': 301  # Uncomment to enable
}
```

### HTTP Caching
The project includes 24-hour HTTP caching to:
- Reduce server load on stat-football.com
- Speed up development and testing
- Respect the target website's resources

## 🔧 Development

### Running Tests
```bash
# Run spider in test mode
scrapy crawl soccer -s LOG_LEVEL=DEBUG
```

### Adding New Leagues
The spider automatically discovers leagues from the website structure. No manual configuration needed.

### Customizing Data Processing
Modify `soccer/pipelines.py` to add custom data processing logic.

## 📈 Usage Examples

### Basic Data Extraction
```python
from scrapy import cmdline
cmdline.execute("scrapy crawl soccer -o matches.json".split())
```

### With Database Storage
1. Configure database settings in `settings.py`
2. Enable database pipeline
3. Run the spider

### Custom Output Format
```bash
# Export to CSV
scrapy crawl soccer -o matches.csv

# Export to JSON
scrapy crawl soccer -o matches.json

# Export to XML
scrapy crawl soccer -o matches.xml
```

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- **Attila Toth** - Original spider implementation ([Scraping Authority](http://scrapingauthority.com))
- **stat-football.com** - Data source
- **Scrapy Community** - Framework and support

## 📞 Support

For questions, issues, or contributions:
- Open an issue on GitHub
- Check the [documentation](docs/)
- Review the [backlog](BACKLOG.md) for planned features

## 🔗 Related Links

- [Scrapy Documentation](https://docs.scrapy.org/)
- [stat-football.com](http://stat-football.com/en/)
- [Dataset on ScrapingHub](https://app.scrapinghub.com/datasets/RS3NYdirkoz)
