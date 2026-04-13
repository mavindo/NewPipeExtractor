# NewPipeExtractor AI Coding Agent Instructions

This guide provides specialized context for developing on the NewPipeExtractor library.

## Architecture & Core Concepts

- **Modular Service Design**: Each streaming service (e.g., YouTube, SoundCloud) has its own package under `org.schabi.newpipe.extractor.services`.
- **Extractor Hierarchy**:
    - `Extractor`: Base class for any content extractor.
    - `StreamExtractor`: For video/audio stream details (e.g., [YoutubeStreamExtractor.java](extractor/src/main/java/org/schabi/newpipe/extractor/services/youtube/extractors/YoutubeStreamExtractor.java)).
    - `ListExtractor`: For pages containing lists of items (playlists, channels, search results).
    - `InfoItemExtractor`: For individual items within a list.
- **Link Handling**: `LinkHandler` and `LinkHandlerFactory` (e.g., [YoutubeStreamLinkHandlerFactory.java](extractor/src/main/java/org/schabi/newpipe/extractor/services/youtube/linkHandler/YoutubeStreamLinkHandlerFactory.java)) manage URL parsing and validation. Do not manually parse URLs; use the appropriate `LinkHandler`.
- **Downloader Interface**: All network requests must go through the `Downloader` interface. Never use `java.net.URL` or other HTTP clients directly in extractors.
- **Localization & Dates**: Use `TimeAgoParser` for relative dates (e.g., "2 hours ago"). Access it via `getTimeAgoParser()` in extractors.
- **Pagination**: Use the `Page` class to handle multi-page results in `ListExtractor` subclasses.

## Data Flow

1. `StreamingService` receives a URL.
2. `LinkHandler` validates the URL and extracts IDs.
3. The appropriate `Extractor` is used to `fetchPage()`.
4. Extractor uses `Downloader` to get raw content (HTML/JSON).
5. Data is mapped to `InfoItem` objects (e.g., `StreamInfoItem`) using `Collector` classes.

## Development Workflows

- **Building**: Use `./gradlew build` to compile all modules.
- **Testing**:
    - Run all tests: `./gradlew :extractor:test`
    - **Downloader Modes**:
        - `MOCK` (default): Uses local files in `extractor/src/test/resources/mocks/`.
        - `RECORDING`: Records real network responses to files for future mock testing.
        - `REAL`: Performs actual network requests.
        - Example: `./gradlew test -Ddownloader=RECORDING`
- **Adding a Service**: Implement `StreamingService`, `LinkHandlerFactory`, and core extractors (`Stream`, `Channel`, `Playlist`).

## Coding Patterns & Conventions

- **Parsing Utilities**: Use static methods in `YoutubeParsingHelper` (and similar service helpers) for common extraction tasks to avoid code duplication.
- **Exception Handling**: Prefer specific exceptions like `ContentNotAvailableException` or `GeographicRestrictionException` over generic `ParsingException`.
- **Mock Testing**: New tests should ideally use `MockDownloader`. Use `./gradlew test -Ddownloader=RECORDING` to generate mocks for new URLs.
- **Nullability**: Use `@Nonnull` and `@Nullable` annotations aggressively to prevent NPEs in a library environment.

## Key Directories

- `extractor/src/main/java/org/schabi/newpipe/extractor/`: Core interfaces and base classes.
- `extractor/src/main/java/org/schabi/newpipe/extractor/services/`: Service-specific implementations.
- `extractor/src/test/resources/mocks/`: Regression test data.
