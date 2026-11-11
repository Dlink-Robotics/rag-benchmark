# rag-benchmark

A hands-on project for testing a RAG (retrieval-augmented generation) system from start to finish:
build a baseline, find which stage is at fault, trust the measurement, and time every stage.

Each step has two halves. A **learn** half (one idea, one analogy) and a **build** half (a small,
reviewable piece of benchmark code). See [docs/roadmap.md](docs/roadmap.md) for all 18 steps.

## The idea in one paragraph

Start with every part of the RAG system at a default setting and measure it once. That is the
baseline. Then change one part at a time and measure again. Because only one part changed, any
movement in the scores points to that part. Retrieval is tested on its own, without a judge,
before answers are judged. Every stage is also timed, so a decision uses quality and speed together.

## The stages being tested

| Stage | When it runs | What it answers |
|---|---|---|
| Split into chunks | once, when documents are added | does a chunk keep the answer together? |
| Turn chunks into vectors | once, when documents are added | does the embedding model place related text close together? |
| Find candidates | every question | is the right chunk in the candidate pool? |
| Rerank | every question | does the right chunk reach the final top few? |
| Write the answer | every question | does the model use the chunks correctly? |
| Judge the answers | only when testing | can the scores be trusted, and are they stable? |

## How we work

- The owner leads the project, writes the roadmap and reviews every pull request.
- Collaborators work on a branch and open a pull request. Nothing goes straight to `main`.
- One change per pull request, with the benchmark numbers (quality and speed) in the description.
- Pin every model to an exact version. A rolling alias can change the judge without notice.
- Never commit API keys, `.env` files or machine-specific paths.

## Status

Nothing is built yet. The next step is level 1 in the roadmap.
