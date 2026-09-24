# Specifications for pi_customization Enhancements

This document outlines the approved specifications for a series of enhancements to the `.pi_customization` script.

## 1. Enhanced Session Management

The goal is to provide robust, user-friendly session recording with meaningful names and true continuation capabilities.

### 1.1. Robust Recording and Naming

This feature changes how new sessions are recorded and named.

-   **End-of-Session Timestamp**: Log files for new, unnamed sessions will be timestamped with the time the session *ends*, not when it starts.
-   **Graceful Failure Handling**: If a session is terminated abruptly (e.g., via `Ctrl+C`), the partial session log will be saved.
    -   A `trap` will catch the exit signal.
    -   The temporary session file will be renamed to include an `_INCOMPLETE` suffix (e.g., `2023-10-27_16-45-00_INCOMPLETE.log`) to preserve the data.
-   **Meaningful Naming**: A flexible, prioritized system will be used to name session logs:
    1.  **`--name` flag**: The user can provide a name upfront (e.g., `pi --name "my-feature"`).
    2.  **Post-Session Prompt**: If no name is provided, the script will prompt the user to enter a name after the session concludes.
    3.  **Timestamp Fallback**: If the user skips the prompt, the end-of-session timestamp will be used as the filename.
-   **Filename Sanitization**: All user-provided names will be sanitized to ensure they are valid filenames.

### 1.2. True Session Continuation

This feature modifies the `--resume` functionality to ensure a seamless workflow.

-   **Append Mode**: When resuming a session, all new activity will be appended to the *original* log file instead of creating a new one. The `script -a` command will be used.
-   **Timestamp Update**: Appending to a file updates its modification timestamp. This ensures that a recently continued session will correctly appear at the top of the list when `pi --resume` is used again, as the listing is sorted by most recently modified.

## 2. Code Refactoring

To improve readability and maintainability, the monolithic `pi()` function will be broken down into smaller, single-responsibility helper functions.

-   **`_pi_sanitize_filename()`**: Cleans a string to be a valid filename.
-   **`_pi_parse_args()`**: Handles the `--name` flag logic.
-   **`_pi_handle_resume()`**: Manages the interactive session resumption menu and logic.
-   **`_pi_run_new_session()`**: Contains all logic for recording a brand-new session.
-   **`_pi_run_resumed_session()`**: Contains the logic for appending to an existing session.
-   **`pi()`**: The main function, which acts as an orchestrator, calling the helper functions in the correct sequence.

## 3. Per-Project Encrypted Provider Management

This feature provides a secure and convenient way to manage different LLM provider credentials for different projects.

### 3.1. Storage and Encryption

-   **Credential File**: Provider data will be stored in a file named `.pi_providers.enc` in the project's root directory.
-   **Encryption**: The file will be encrypted using `openssl aes-256-cbc`.
-   **Password-Based Access**:
    -   Access requires a password that is prompted for at the start of a session.
    -   The password and any decrypted API keys are **only held in memory** for the duration of the script's execution and are never stored on disk.

### 3.2. User Workflows

-   **First-Time Initialization**:
    1.  If `.pi_providers.enc` is not found, the script will offer to create it.
    2.  It will prompt the user to set and confirm a new password for the file.
    3.  It will guide the user through adding their first provider and key.
-   **Standard Execution**:
    1.  If `.pi_providers.enc` is found, the script will prompt for the password to decrypt it.
    2.  It will display an interactive menu with the following choices:
        -   A numbered list of all saved providers.
        -   An option to `[Add a new provider]`.
        -   An option to `[Continue without a provider]`.
-   **Credential Injection**: Once a provider is selected, the script will inject the necessary `--provider` and `--api-key` arguments into the `pi` command before executing it, ensuring the entire session uses the correct credentials.
