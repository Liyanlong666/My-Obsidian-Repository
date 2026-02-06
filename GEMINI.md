# GEMINI.md

## Directory Overview
This directory is a comprehensive **Obsidian Knowledge Base** primarily focused on Java development, software engineering interview preparation ("八股文"), and specialized database technologies. It serves as a centralized hub for learning notes, architectural deep dives, and technical references.

The repository is structured for ease of navigation within Obsidian, utilizing nested folders for categorization and specific `.assert` directories to store associated visual assets (images/diagrams).

## Key Directories & Files
- **`JAVA/`**: The core content area.
    - **`八股文/`**: High-quality, detailed interview notes covering JVM, JUC (Concurrency), MySQL, Redis, and Computer Networking.
    - **`时序数据库InfluxDB/`**: Specialized notes on InfluxDB architecture, TSM storage engines, and performance benchmarks.
    - **`kafka/`**: Deep dives into Apache Kafka components and logic.
    - **`algorithm-go/`**: Algorithm practice and notes implemented in Go.
    - **`AI Coding Agent Prompt/`**: Research or notes on AI-assisted coding prompts.
- **`.obsidian/`**: Configuration files for the Obsidian app, including plugin settings and workspace layouts.
    - **`plugins/obsidian-git/`**: Configuration for the Git integration plugin, enabling automatic backups and version control.
- **`README.md`**: A brief introductory file for the repository.

## Usage
- **Note Taking & Reading**: Best viewed using [Obsidian](https://obsidian.md/). Use the folder structure to navigate specific topics.
- **Version Control**: The project is pre-configured with `obsidian-git`. Changes should be managed via Git to ensure history is preserved.
- **Image Assets**: Most Markdown files reference images stored in adjacent `.assert` or `*.assert` folders. Do not move or rename these folders to avoid broken links.
- **Knowledge Synthesis**: The repository follows a pattern of aggregating high-quality online resources (like "沉默王二", "小林coding", etc.) into a cohesive, searchable personal library.

## Development & Maintenance
- **Markdown Style**: Adheres to GitHub-flavored Markdown. Notes often include callouts, code blocks, and embedded images.
- **Reorganization Logic**: When organizing notes, prefer grouping by functional area (e.g., "JVM", "MySQL") and merging redundant snippets into comprehensive guides.
