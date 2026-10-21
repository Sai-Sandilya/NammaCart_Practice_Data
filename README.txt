NAMMACART PRACTICE DATA
Version 1.0 - all data is synthetic

1,179,496 physical records; 100,000 base orders + 7,000 later orders.
No solution code, SQL answers, ETL scaffolding or generator is included.

START HERE
1. Read the included Project Brief and Sprints PDF.
2. Preserve this extracted source folder unchanged.
3. Create your own code repository and project structure.
4. Begin with hand_check, then a related small subset, then the full baseline.
5. Process incremental batches in July 1-7 arrival order.
6. Treat challenge_packs as separate experiments, never normal source tables.

DATA ORIENTATION
ONE DATASET, SEVERAL RELATED TABLES
A realistic project needs several tables because orders, items, payments, shipments and customer changes have different row meanings. They are delivered as one download.

BASELINE
100,000 logical orders placed from January 1 through June 20, 2025 UTC, with source activity available before July 1, 2025 UTC. Customer signups occur in 2024. Inventory is a June 2025 sample.

SEVEN LATER BATCHES
batch_date=2025-07-01 through 2025-07-07 each introduces 1,000 new logical orders and related records. Their availability cutoff is the next UTC midnight. In total, 350 orders have old business dates but arrive in these later batches.

CUSTOMER CHANGES
customer_cdc exists only in incremental folders. Changes include new customers, full-row updates, soft deletes, duplicates and retroactive corrections. Do not treat a late-arriving effective timestamp as a newer current version automatically.

PHYSICAL VERSUS LOGICAL COUNTS
The manifest counts physical records, including deliberate duplicates and malformed JSON lines. There are 107,000 unique order IDs after all deliveries. Counts of accepted business rows depend on the specified cleaning and eligibility rules.

FILE ARRIVAL DATE
Folder batch_date is an arrival-batch label. It is not the customer signup date, order date, payment date or report date. Late records deliberately break that assumption.

FILE PARTS
CSV files have a header in every part and use UTF-8. JSON Lines contains one event per line; 21 source lines are deliberately malformed. File splitting is for transfer and processing, not a business partitioning guarantee.

ZERO ACTIVITY
Some batch/table combinations have no records and therefore no file. Use the manifest to distinguish zero expected activity from an expected file that is missing. Do not assume every source must produce rows every day.

SEPARATE TEST PACKS
challenge_packs and hand_check are isolated exercises. They are not part of the normal business load. Several challenge files contain case descriptions or smaller schemas, so read their instructions before writing an adapter.

SYNTHETIC DATA
All people, suppliers, products and activity are invented. Email addresses use example.invalid. Do not contact them. The data is useful for engineering practice, not evidence about real customers or real market behavior.

MANIFEST
manifest.json lists every data CSV/JSONL path, byte size, SHA-256 and physical record count. It does not hash itself, the PDF or documentation. CSV header rows are excluded. Malformed JSON physical lines are included.
table_columns.json lists the flat CSV column order; the full dictionary explains nested events and CDC.

FILE LOCATIONS
baseline/ : initial source delivery
incremental/batch_date=2025-07-DD/ : later delivery
hand_check/ : nine simple records with independently calculated expected values; deliberately reduced schema
challenge_packs/ : isolated incident records and case descriptions
challenge_packs/stream_sequence/ : deliver files 01 through 07 individually in order

STREAM FIXTURE
Use one file per micro-batch and wait for completion between files. With a one-hour watermark, W2 and W3 advance event time before W4 arrives far behind it. W5 is inside the current late-data allowance when first delivered. W6 advances time further; file 07 is a replay of W5 and is now old as well as duplicated. Inspect actual query progress and output mode rather than assuming all records must appear in final aggregates. Run deduplication and window-count experiments separately if your Spark query restrictions require it.

CHALLENGE PACK SCHEMAS
schema_break.jsonl and stream_sequence files resemble event records. conflicting_customer_changes.jsonl contains CDC conflicts. amount_and_quantity_errors.jsonl and relational_edge_cases.jsonl are named case descriptions with deliberately smaller schemas; build tiny tests or adapters around them. Header-only and wrong-header CSVs test file intake.

TYPES AND MONEY
CSV values are text on disk. Interpret target types only after preserving raw values. Money is integer INR paise (100 paise = 1 rupee). Timestamps are UTC ISO-8601; reporting dates use Asia/Kolkata.

NORMAL BATCH POLICY
No data file for a table can mean zero activity if the manifest expects none. A missing path listed in manifest.json is a failed delivery. Use the distinction in your ingestion and scheduling design.

HAND-CHECK EXPECTATIONS
H1 merchandise 28,000 paise; approved return 9,000; net merchandise 19,000; net cash 24,000; shipping 5,000. H2 is cancelled and excluded. These are acceptance values, not query solutions.
