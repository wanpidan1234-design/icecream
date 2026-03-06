# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

A Python CLI application for managing an ice cream shop — inventory, orders, and sales reporting. Built with Python 3.11+, SQLite, and Click.

## Development Commands

- **Install**: `pip install -e ".[dev]"`
- **Run tests**: `pytest`
- **Run single test**: `pytest tests/test_shop.py::test_place_order_success -v`
- **Run tests with coverage**: `pytest --cov=icecream`
- **Lint**: `ruff check src/ tests/`
- **Format**: `ruff format src/ tests/`
- **Run CLI**: `icecream --help`

Ruff config: line length 100, rules E/F/I/N/W/UP (in pyproject.toml).

## Architecture

The app follows a three-layer architecture: **CLI → Business Logic → Data Layer**.

- `cli.py` — Click command group (`add`, `flavors`, `order`, `stock`, `report`, `orders`). Each command calls into `shop.py` or `reports.py`, never directly into `models.py`.
- `shop.py` — Validation and business rules. Raises `ShopError` on invalid input. Functions: `create_flavor()`, `place_order()`, `restock()`.
- `models.py` — SQLite operations and dataclasses (`Flavor`, `Order`). All SQL lives here. Uses WAL mode and stores monetary values as `Decimal` text strings to avoid float precision issues.
- `reports.py` — Read-only reporting queries (`total_revenue()`, `sales_by_flavor()`, `top_flavor()`). Operates directly on the database, not through `shop.py`.

**Database**: SQLite with two tables (`flavors`, `orders`). Default path: `icecream.db` in cwd. `get_connection(db_path)` accepts an optional path override (used by tests with `tmp_path` fixtures).

## Testing

Tests use a `conn` fixture from `conftest.py` that provides an isolated temporary SQLite database per test. Pass this connection to all functions under test — no shared state between tests.

## Git Workflow

- Branch from `main`: `git checkout -b feature/<name>`
- Run `ruff check` and `pytest` before every commit
- Imperative commit messages ("Add feature", not "Added feature")
- No force-push or `reset --hard` without explicit approval

## Custom Slash Commands

- `/git-summary` — Git state overview
- `/git-review` — Review uncommitted changes for bugs/style/security
- `/git-smart-commit` — Lint + test + commit (refuses broken code)
- `/git-pr` — Push and create pull request
- `/git-branch-cleanup` — Delete merged branches

## Conventions

- Type hints on all function signatures
- Functions max ~30 lines
- SQL queries in `models.py`, business logic in `shop.py`
- `decimal.Decimal` for all monetary values
