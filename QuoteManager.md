# Quote Management CLI Application

**Purpose:** This conceptual program provides a command-line interface for managing a collection of textual quotes. It supports core CRUD (Create, Read, Update, Delete) operations, search, random selection, and basic rating functionality, with data persistence to a file. This project demonstrates principles of interactive command-line interface design, data structuring, file I/O, and user input validation.

---

## I. Main Program Flow

1.  **Constants & Global State Initialization:**
    * `DATA_STORAGE_FILE`: Constant representing the filename for data persistence (e.g., 'quotes_data.json').

2.  **Data Loading & Error Handling:**
    * Attempt to open `DATA_STORAGE_FILE` with read permissions.
    * Attempt to load its contents into `quote_collection` (expected JSON format).
    * **Exception Handling:** If file is not found, corrupted, or not valid JSON:
        ```
        - Initialize `quote_collection` as an empty list (to start fresh).
        - Display a message indicating no previous data was loaded or an error occurred during loading.
        ```

3.  **Command-Line Interface (CLI) Main Loop:**
    * Display a welcome message to the user.
    * Enter an indefinite loop to repeatedly prompt for user commands until an exit command is given.
    * **Prompt for Action:** Display a menu of available commands:
        * `[a]dd`: Add a new quote.
        * `[l]ist`: Display all quotes.
        * `[s]earch`: Find quotes by keyword.
        * `[v]iew`: Display details of a specific quote by index.
        * `[r]andom`: Display a random quote.
        * `[d]elete`: Remove a quote by index.
        * `[q]uit`: Exit the application.
    * Inform user about optional "expedited" input (e.g., "v 5" to view index 5 directly).
    * Capture `raw_user_input` and convert to lowercase.

4.  **Parse & Validate `raw_user_input`:**
    * Split `raw_user_input` by the first space to separate `command_prefix` and `optional_parameter`.
    * If `optional_parameter` exists and is not empty, store it. Otherwise, set it to `None`.
    * **Type Conversion & Validation for Indexed Commands:**
        * For `view` or `delete` commands, if `optional_parameter` is provided:
            * Attempt to convert `optional_parameter` to an integer (`parsed_index`).
            * **Error Handling:** If `parsed_index` is not a valid integer:
                ```
                - Display an "invalid index format" error message.
                - Continue to the next loop iteration (re-prompt user).
                ```
        * For other commands, no specific parameter type conversion is needed at this stage.

5.  **Pre-check for Empty Collection (for non-add/quit commands):**
    * If `quote_collection` is empty AND `command_prefix` is not 'a' or 'q':
        ```
        - Display "No quotes saved in the catalogue" message.
        - Continue to the next loop iteration.
        ```

6.  **Process `command_prefix` Actions:**

    * **If `command_prefix` is 'a' (Add Quote):**
        ```
        - Create `new_quote_record` as a dictionary/object. Initialize `likes` and `loves` fields to 0.
        - Prompt user for "Quote Text" (using `get_non_empty_string_input`). Store value in `new_quote_record['quote']`.
        - Prompt user for "Author Name" (using `get_non_empty_string_input`). Store value in `new_quote_record['author']`.
        - Prompt user for "Year of Quote" (optional, allow blank). Store result as `new_quote_record['year']`.
        - If 'year' input is blank, remove `new_quote_record['year']` field.
        - Append `new_quote_record` to `quote_collection`.
        - Call `save_collection_to_file` function with `quote_collection`.
        - Display "Quote added successfully" message.
        ```

    * **Else If `command_prefix` is 'l' (List Quotes):**
        ```
        - For each `quote_record` in `quote_collection` (with its `index`):
            - Call `display_formatted_quote_info` function, passing its index, `quote_record`, and `True` (for abbreviated display).
        ```

    * **Else If `command_prefix` is 's' (Search Quotes):**
        ```
        - If `optional_parameter` exists, set `search_query` to its value.
        - Otherwise, prompt user for "Search Term" (using `get_non_empty_string_input`), convert to lowercase, store as `search_query`.
        - Initialize `results_found` flag to `False`.
        - For each `quote_record` in `quote_collection` (with its `index`):
            - Check if `search_query` is present in any relevant fields of `quote_record` (e.g., 'quote', 'author', 'year').
            - If `search_query` is found:
                - Set `results_found` to `True`.
                - Call `display_formatted_quote_info` function, passing `index`, `quote_record`, and `True` (for abbreviated display).
        - If `results_found` is `False` after iterating through all quotes:
            - Display "Search value not found" message.
        ```

    * **Else If `command_prefix` is 'v' (View Quote by Index):**
        ```
        - Get `target_index` (from `parsed_index` if expedited, else prompt user using `get_valid_integer_input`).
        - **Attempt Access & Handle Errors:**
            - If `target_index` is less than 1 (conceptual out-of-bounds):
                - Raise an "index out of range" error.
            - Else if `target_index` is a valid index for `quote_collection`:
                - Call `display_formatted_quote_info` function, passing `target_index`, the corresponding `quote_record`, and `False` (for full display).
                - Call `process_rating_display` function, passing the `quote_record` at `target_index`.
            - **Exception Handler:** If an "invalid index" error occurs:
                - Display "Error: Index number not found" message.
        ```

    * **Else If `command_prefix` is 'd' (Delete Quote by Index):**
        ```
        - Get `target_index_to_delete` (from `parsed_index` if expedited, else prompt user using `get_valid_integer_input`).
        - **Attempt Access & Handle Errors:**
            - If `target_index_to_delete` is less than 1 (conceptual out-of-bounds):
                - Raise an "invalid index" error.
            - Else if `target_index_to_delete` is a valid index for `quote_collection`:
                - Remove the `quote_record` at `target_index_to_delete` from `quote_collection`.
                - Call `save_collection_to_file` function with the updated `quote_collection`.
                - Display "Quote successfully removed" message.
            - **Exception Handler:** If an "invalid index" error occurs:
                - Display "Index number not found" message.
        ```

    * **Else If `command_prefix` is 'r' (Random Quote):**
        ```
        - Initialize `random_quote_index` with a random integer between 1 and the total number of quotes in `quote_collection`.
        - If `random_quote_index` corresponds to a valid index in `quote_collection`:
            - Call `display_formatted_quote_info` function, passing `random_quote_index`, the corresponding `quote_record`, and `False` (for full display).
            - Call `process_rating_display` function, passing the `quote_record` at `random_quote_index`.
        ```

    * **Else If `command_prefix` is 'q' (Quit Program):**
        ```
        - Display a "Goodbye" message.
        - Exit the main interaction loop to terminate the program.
        ```

    * **Else (Invalid Command):**
        ```
        - Display an "Invalid command - please try again" message.
        ```

7.  **Final Program Statistics Display (After Loop Exit):**
    * Display a final heading "Program Session Statistics".
    * Display total number of quotes processed.
    * Display total actions that did not involve rating changes (e.g., listing, searching, viewing).
    * Display total rating changes recorded.

---

## II. Supporting Functions

1.  **`display_framed_heading(heading_text, frame_width)`:**
    * **Parameters:** `heading_text` (string), `frame_width` (integer, default).
    * **Action:** Prints a text-based framed box around the `heading_text`, centered within the specified `frame_width`. (e.g., using generic text symbols like `-`, `|`, `+` to form the frame).

2.  **`get_valid_integer_input(prompt_message, error_message)`:**
    * **Parameters:** `prompt_message` (string for user), `error_message` (string for validation failure).
    * **Action:** Prompts the user for input. Continuously re-prompts with `error_message` until a valid integer (e.g., greater than or equal to 1 for an index) is entered. Returns the validated integer.

3.  **`get_non_empty_string_input(prompt_message, error_message)`:**
    * **Parameters:** `prompt_message` (string for user), `error_message` (string for validation failure).
    * **Action:** Prompts the user for input. Continuously re-prompts with `error_message` until a non-empty string (after stripping leading/trailing whitespace) is entered. Returns the validated string.

4.  **`save_collection_to_file(data_collection)`:**
    * **Parameters:** `data_collection` (list of dictionaries/objects).
    * **Action:** Opens `DATA_STORAGE_FILE` with write permissions. Writes `data_collection` content to the file in JSON format. Closes the file.

5.  **`display_formatted_quote_info(index, quote_record, is_abbreviated_display)`:**
    * **Parameters:** `index` (integer), `quote_record` (dictionary/object representing a quote), `is_abbreviated_display` (boolean flag).
    * **Action:** Displays formatted quote information:
        * If `is_abbreviated_display` is `True`: Shows "index) 'quote_text' - author" with `quote_text` truncated if long, followed by "...". Handles cases where 'year' is missing.
        * If `is_abbreviated_display` is `False`: Shows "index) 'quote_text'\n- author, year" (full text). Handles cases where 'year' is missing.

6.  **`process_rating_display(quote_record_entry)`:**
    * **Parameters:** `quote_record_entry` (a single quote dictionary/object).
    * **Action:** Calculates an `overall_rating` based on 'likes' (1 point each) and 'loves' (2 points each). Displays the number of likes and loves (with correct pluralization), along with the `overall_rating`. If no ratings, states the quote has not been rated.

---

**Disclaimer:** This pseudocode is provided for illustrative purposes to demonstrate algorithmic thinking, data management, and problem-solving skills acquired during university coursework. It is a conceptual representation and not directly executable production code. All specific university course identifiers or proprietary elements have been removed or generalised to adhere to academic integrity policies.