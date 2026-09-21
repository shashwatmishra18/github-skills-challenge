# AIOps Pipeline Assessment

This repository contains a small Python AIOps pipeline that reads service telemetry,
detects threshold and log anomalies, publishes anomaly events to an in-memory topic,
and consumes the published events.

## Verified Findings

The operational data contains 10 records for `payment-service`. Two records are
anomalous:

- `2026-09-20T10:05:00`: high response time and an `ERROR` log.
- `2026-09-20T10:06:00`: high response time, CPU, memory, and an `ERROR` log.

The original workflow detected both anomalies but consumed zero events because the
producer and consumer were connected to separate `EventTopic` instances. The
detector also labeled `WARNING` records as error logs.

## Fixes

- The producer and consumer now share one `anomaly-events` topic, so the workflow is
	`EventProducer -> EventTopic -> EventConsumer`.
- Only records with log level `ERROR` receive the `Error log detected` reason.

## Verified Results

Running the CLI after the fixes produced:

```text
Records processed: 10
Anomalies detected: 2
Events consumed: 2
```

The consumed events are the same two events detected from the input data. The
complete test suite also passes: `8 passed`.

## Reproduction

From the repository root:

```bash
python3 src/aiops_pipeline.py
python3 -m pytest -q
```

The repository's `pytest` executable alone may not include the repository root on
`sys.path`; use `python3 -m pytest -q` for the verified test command.

## Limitations

- `EventTopic` is an in-memory simulation; events do not persist between processes.
- Detection uses fixed response-time, CPU, and memory thresholds from
	`AnomalyDetector`.
- No assessment PDF was present in the Codespace during this assessment, so the
	findings above are based on the checked-in source, data, tests, and observed runs.

---

&copy; 2025 GitHub &bull; [Code of Conduct](https://www.contributor-covenant.org/version/2/1/code_of_conduct/code_of_conduct.md) &bull; [MIT License](https://gh.io/mit)

