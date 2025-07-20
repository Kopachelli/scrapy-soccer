# Contributing to Soccer Data Scraper

Thank you for your interest in contributing to the Soccer Data Scraper project! This document provides guidelines and information for contributors.

## Table of Contents

1. [Getting Started](#getting-started)
2. [Development Setup](#development-setup)
3. [Code Style](#code-style)
4. [Testing](#testing)
5. [Pull Request Process](#pull-request-process)
6. [Issue Reporting](#issue-reporting)
7. [Feature Requests](#feature-requests)
8. [Documentation](#documentation)

## Getting Started

### Prerequisites

- Python 3.7+
- Git
- pip
- MySQL (optional, for database testing)

### Fork and Clone

1. **Fork the repository**
   - Go to [https://github.com/zseta/scrapy-soccer](https://github.com/zseta/scrapy-soccer)
   - Click the "Fork" button in the top right

2. **Clone your fork**
```bash
git clone https://github.com/YOUR_USERNAME/scrapy-soccer.git
cd scrapy-soccer
```

3. **Add upstream remote**
```bash
git remote add upstream https://github.com/zseta/scrapy-soccer.git
```

## Development Setup

### Environment Setup

1. **Create virtual environment**
```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

2. **Install dependencies**
```bash
pip install -r requirements.txt
pip install -r requirements-dev.txt  # Development dependencies
```

3. **Install pre-commit hooks**
```bash
pre-commit install
```

### Development Dependencies

Create `requirements-dev.txt`:
```txt
# Testing
pytest>=6.0.0
pytest-scrapy>=1.2.0
pytest-cov>=2.10.0
pytest-mock>=3.3.0

# Code Quality
black>=21.0.0
flake8>=3.9.0
isort>=5.9.0
mypy>=0.910

# Documentation
sphinx>=4.0.0
sphinx-rtd-theme>=0.5.0

# Development Tools
pre-commit>=2.15.0
jupyter>=1.0.0
ipython>=7.0.0
```

### Pre-commit Configuration

Create `.pre-commit-config.yaml`:
```yaml
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.2.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: check-added-large-files

  - repo: https://github.com/psf/black
    rev: 22.3.0
    hooks:
      - id: black

  - repo: https://github.com/pycqa/isort
    rev: 5.10.1
    hooks:
      - id: isort

  - repo: https://github.com/pycqa/flake8
    rev: 4.0.1
    hooks:
      - id: flake8
```

## Code Style

### Python Style Guide

We follow [PEP 8](https://www.python.org/dev/peps/pep-0008/) with some modifications:

- **Line length**: 88 characters (Black default)
- **Indentation**: 4 spaces
- **String quotes**: Double quotes (Black default)
- **Import sorting**: isort configuration

### Code Formatting

1. **Automatic formatting**
```bash
# Format code with Black
black soccer/

# Sort imports
isort soccer/
```

2. **Linting**
```bash
# Run flake8
flake8 soccer/

# Type checking
mypy soccer/
```

### Naming Conventions

- **Classes**: PascalCase (e.g., `MatchSpider`)
- **Functions and variables**: snake_case (e.g., `clean_match_data`)
- **Constants**: UPPER_SNAKE_CASE (e.g., `MAX_RETRIES`)
- **Files**: snake_case (e.g., `match_spider.py`)

### Documentation

- **Docstrings**: Use Google style docstrings
- **Comments**: Explain complex logic, not obvious code
- **README**: Keep updated with new features

Example docstring:
```python
def clean_match_data(match_data: dict) -> dict:
    """Clean and validate match data.
    
    Args:
        match_data: Raw match data dictionary
        
    Returns:
        Cleaned match data dictionary
        
    Raises:
        ValueError: If required fields are missing
    """
    # Implementation here
```

## Testing

### Test Structure

```
tests/
├── __init__.py
├── conftest.py
├── test_spiders/
│   ├── __init__.py
│   └── test_match_spider.py
├── test_pipelines/
│   ├── __init__.py
│   └── test_cleaning_pipeline.py
└── test_integration/
    ├── __init__.py
    └── test_full_scraping.py
```

### Writing Tests

1. **Unit Tests**
```python
# test_pipelines/test_cleaning_pipeline.py
import pytest
from soccer.pipelines import CleaningPipeline

class TestCleaningPipeline:
    def setup_method(self):
        self.pipeline = CleaningPipeline()
    
    def test_clean_date_valid(self):
        """Test date cleaning with valid input."""
        result = self.pipeline.clean_date("25.12.2023")
        assert result == "2023-12-25"
    
    def test_clean_date_invalid(self):
        """Test date cleaning with invalid input."""
        with pytest.raises(ValueError):
            self.pipeline.clean_date("invalid-date")
```

2. **Integration Tests**
```python
# test_integration/test_full_scraping.py
import pytest
from scrapy.crawler import CrawlerProcess
from scrapy.utils.project import get_project_settings

def test_full_scraping_workflow():
    """Test complete scraping workflow."""
    settings = get_project_settings()
    settings.set('ITEM_PIPELINES', {
        'soccer.pipelines.CleaningPipeline': 300,
    })
    
    process = CrawlerProcess(settings)
    process.crawl('soccer')
    process.start()
    
    # Assertions here
```

3. **Fixtures**
```python
# conftest.py
import pytest
from soccer.items import MatchItem

@pytest.fixture
def sample_match_item():
    """Provide a sample match item for testing."""
    return MatchItem(
        country="England",
        league="Premier League",
        date="25.12.2023",
        home_team="Manchester United",
        away_team="Liverpool",
        home_goals1="1",
        home_goals2="1",
        away_goals1="0",
        away_goals2="1"
    )

@pytest.fixture
def mock_response():
    """Provide a mock response for testing."""
    from scrapy.http import HtmlResponse
    from scrapy.http import Request
    
    return HtmlResponse(
        url="http://stat-football.com/test",
        body=b"<html><body>Test content</body></html>",
        request=Request("http://stat-football.com/test")
    )
```

### Running Tests

```bash
# Run all tests
pytest

# Run with coverage
pytest --cov=soccer

# Run specific test file
pytest tests/test_pipelines/test_cleaning_pipeline.py

# Run with verbose output
pytest -v

# Run tests in parallel
pytest -n auto
```

### Test Coverage

We aim for at least 80% test coverage. Generate coverage report:

```bash
pytest --cov=soccer --cov-report=html
open htmlcov/index.html  # View coverage report
```

## Pull Request Process

### Before Submitting

1. **Update your fork**
```bash
git fetch upstream
git checkout main
git merge upstream/main
```

2. **Create feature branch**
```bash
git checkout -b feature/your-feature-name
```

3. **Make changes**
   - Write code following style guidelines
   - Add tests for new functionality
   - Update documentation

4. **Test your changes**
```bash
# Run tests
pytest

# Run linting
flake8 soccer/
black --check soccer/
isort --check-only soccer/

# Run type checking
mypy soccer/
```

5. **Commit your changes**
```bash
git add .
git commit -m "feat: add new feature description

- Detailed description of changes
- Related issue: #123"
```

### Commit Message Format

We use [Conventional Commits](https://www.conventionalcommits.org/):

- `feat:` New feature
- `fix:` Bug fix
- `docs:` Documentation changes
- `style:` Code style changes
- `refactor:` Code refactoring
- `test:` Test changes
- `chore:` Maintenance tasks

### Submitting PR

1. **Push to your fork**
```bash
git push origin feature/your-feature-name
```

2. **Create Pull Request**
   - Go to your fork on GitHub
   - Click "New Pull Request"
   - Select your feature branch
   - Fill out the PR template

3. **PR Template**
```markdown
## Description
Brief description of changes

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation update

## Testing
- [ ] Unit tests pass
- [ ] Integration tests pass
- [ ] Manual testing completed

## Checklist
- [ ] Code follows style guidelines
- [ ] Self-review completed
- [ ] Documentation updated
- [ ] Tests added/updated
- [ ] No breaking changes

## Related Issues
Closes #123
```

### Review Process

1. **Automated checks** must pass
2. **Code review** by maintainers
3. **Address feedback** if needed
4. **Merge** when approved

## Issue Reporting

### Bug Reports

Use the bug report template:

```markdown
## Bug Description
Clear description of the bug

## Steps to Reproduce
1. Step 1
2. Step 2
3. Step 3

## Expected Behavior
What should happen

## Actual Behavior
What actually happens

## Environment
- OS: [e.g., Ubuntu 20.04]
- Python: [e.g., 3.9.0]
- Scrapy: [e.g., 2.5.0]

## Additional Information
Screenshots, logs, etc.
```

### Feature Requests

Use the feature request template:

```markdown
## Feature Description
Clear description of the feature

## Use Case
Why this feature is needed

## Proposed Solution
How you think it should work

## Alternatives Considered
Other approaches you considered

## Additional Information
Mockups, examples, etc.
```

## Documentation

### Documentation Standards

- **README**: Project overview and quick start
- **API docs**: Complete API reference
- **Architecture docs**: System design and components
- **Deployment docs**: Setup and deployment guides
- **Contributing docs**: This file

### Writing Documentation

1. **Use clear, concise language**
2. **Include code examples**
3. **Keep it up to date**
4. **Use proper formatting**

### Documentation Structure

```
docs/
├── README.md
├── ARCHITECTURE.md
├── API.md
├── DEPLOYMENT.md
├── CONTRIBUTING.md
├── CHANGELOG.md
└── examples/
    ├── basic_usage.py
    └── advanced_usage.py
```

## Code of Conduct

### Our Standards

- **Be respectful** and inclusive
- **Be collaborative** and constructive
- **Be professional** in communications
- **Be helpful** to newcomers

### Enforcement

- Report violations to maintainers
- Maintainers will address issues promptly
- Violations may result in temporary or permanent ban

## Getting Help

### Resources

- **Documentation**: [docs/](docs/)
- **Issues**: [GitHub Issues](https://github.com/zseta/scrapy-soccer/issues)
- **Discussions**: [GitHub Discussions](https://github.com/zseta/scrapy-soccer/discussions)

### Contact

- **Maintainers**: [@zseta](https://github.com/zseta)
- **Email**: [project-email@example.com](mailto:project-email@example.com)

## Recognition

Contributors will be recognized in:

- **README.md** contributors section
- **CHANGELOG.md** for significant contributions
- **Release notes** for major features

Thank you for contributing to Soccer Data Scraper! 🚀