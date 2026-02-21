# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **quantum pulse waveform generation library** used for quantum computing control systems. It provides a declarative way to define, manipulate, and sample waveforms for arbitrary waveform generators (AWGs) in quantum experiments.

## Common Commands

### Installation
```bash
pip install -e .                    # Editable installation
```

### Testing
```bash
pytest                              # Run all tests
pytest tests/test_waveform.py       # Run specific test file
pytest -v                           # Verbose output
```

### Build Cython Extension
```bash
python setup.py build_ext --inplace  # Build Cython extension in place
```

## Architecture

### Core Components

**Waveform Class** (`waveforms/waveform.py`): The main class representing piecewise mathematical functions. Each waveform consists of:
- `bounds`: Time boundaries for each segment
- `seq`: Sequence of mathematical expressions for each time segment
- Support for sampling, filtering, serialization, and LaTeX rendering

**Cython Extension** (`waveforms/_waveform.pyx`): Performance-critical implementation of:
- Base wave functions (LINEAR, GAUSSIAN, COS, SINC, EXP, DRAG, MOLLIFIER, etc.)
- Waveform arithmetic (add, mul, shift, pow)
- Expression simplification and filtering
- `calc_parts()` for efficient evaluation at sample points

**Expression Parser** (`waveforms/waveform_parser.py` + `Waveform.g4`):
- ANTLR4 grammar for parsing waveform expressions from strings
- `wave_eval()` function evaluates string expressions like `"gaussian(10) >> 5 + square(20) * cos(2*pi*23.1)"`

### Waveform Operations

- **Arithmetic**: `+`, `-`, `*`, `/`, `**` (power)
- **Time shift**: `>>` (right shift/delay), `<<` (left shift/advance)
- **Derivative**: `D(wav, d)` for d-th derivative
- **Mixing**: `mixing(I, Q, freq, phase)` for IQ modulation

### Pulse Shapes

Common shapes: `gaussian()`, `square()`, `cosPulse()`, `hanning()`, `coshPulse()`, `drag()`, `chirp()`, `mollifier()`, `step()`, `sinc()`

### Serialization

Waveforms can be serialized via `tolist()`/`fromlist()` and `totree()`/`fromtree()` for storage and inter-process communication.

## Code Conventions

- Type hints are used throughout
- Cython extension uses typed memoryviews for numpy array operations
- The internal representation uses nested tuples: `((mul_terms,), (amplitudes,))` where each mul_term is `((func_tuple,), (exponents,))`
