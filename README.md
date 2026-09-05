# G-ForceZero

G-ForceZero is a highly optimized, UCI-compatible neural network chess engine written in C++. It features a custom native C++ NNUE trainer and an end-to-end self-play pipeline for generating its own training data and improving its evaluation.

## Project Structure

The project has been radically simplified to separate the engine core from the training tools:

```text
G-ForceZero/
├── src/
│   ├── engine/       # Core engine, UCI interface, search algorithms, and syzygy probing
│   └── tools/        # Neural network trainer, self-play data generator, and utilities
├── data/             # Neural networks (.nnue), opening books, and generated self-play data
├── scripts/          # Helper scripts
└── CMakeLists.txt    # Build system
```

## Compilation

G-ForceZero uses CMake. To compile both the engine (`G-ForceZero`) and the training utility (`trainer`), run:

```bash
mkdir -p build && cd build
cmake ..
make -j$(nproc)
```

The compiled binaries will be placed in the project root directory.

### GPU Support (CUDA)

If you have an NVIDIA GPU and the CUDA Toolkit installed (`nvcc`), CMake will automatically detect it and compile the GPU-accelerated trainer (`trainCudaNetwork`). If `nvcc` is not in your path, you can specify it explicitly:

```bash
CUDACXX=/usr/local/cuda/bin/nvcc cmake ..
```

## Running the Engine

There are three main ways to run G-ForceZero locally:

### 1. Interactive / UCI Mode
Run the engine directly from the command line to interact with it using standard UCI commands:

```bash
./G-ForceZero
```

Common UCI commands:
- `uci` - Initialize engine and display options.
- `isready` - Verify engine readiness.
- `position startpos moves e2e4 c7c5` - Set up a board position.
- `go depth 10` or `go movetime 3000` - Start calculation (for a specified depth or milliseconds).
- `quit` - Exit the engine.

### 2. Chess GUI Integration (Cute Chess, Arena, etc.)
To play against G-ForceZero visually or run engine matches:
1. Install a UCI-compatible GUI such as **Cute Chess** or **Arena**.
2. Add a new engine in the GUI settings:
   - **Engine Name:** `G-ForceZero`
   - **Binary Path:** `/path/to/gforce/G-ForceZero`
   - **Working Directory:** `/path/to/gforce` (to access neural network and data files).

### 3. Running as a Lichess Bot Locally
You can connect G-ForceZero to play live games automatically on Lichess using [`lichess-bot`](https://github.com/lichess-bot-devs/lichess-bot):

1. **Create Bot Account & API Token:**
   - Register a dedicated account on [Lichess.org](https://lichess.org) and upgrade it at [lichess.org/upgrade-bot](https://lichess.org/upgrade-bot).
   - Generate a Personal Access Token at [lichess.org/account/oauth/token](https://lichess.org/account/oauth/token) with the **"Play games with the bot API"** scope enabled.

2. **Set up `lichess-bot`:**
   ```bash
   git clone https://github.com/lichess-bot-devs/lichess-bot.git
   cd lichess-bot
   pip install -r requirements.txt
   ```

3. **Configure `config.yml`:**
   Copy the provided template configuration:
   ```bash
   cp ../render/config.yml ./config.yml
   ```
   Edit `config.yml`:
   - Replace `"YOUR_API_TOKEN"` with your Lichess API Token.
   - Update `engine.dir` to point to your absolute repository path (e.g. `/path/to/gforce`).

4. **Start the Bot:**
   ```bash
   python3 lichess-bot.py
   ```

## Training Pipeline

G-ForceZero comes with a complete self-play and training pipeline. The workflow consists of three simple steps:

### 1. Generate Self-Play Data
Generate raw binary training data by having the engine play against itself using a given opening book (e.g. `openings.epd`):
```bash
./trainer selfplay data/openings.epd --threads 4
```
Games are saved to `data/selfplayGames/`. Press `Ctrl+C` at any time to safely stop the process.

### 2. Prepare Training Data
Convert the raw games into shuffled, parsed training binaries ready for the neural network:
```bash
./trainer prepareTrainingData
```
This extracts positions into `data/trainingData/`.

### 3. Train the Network
Train the network using the prepared data:
```bash
# For CPU training
./trainer trainNetwork

# For GPU training (if compiled with CUDA)
./trainer trainCudaNetwork
```
The trainer will output updated `.nnue` files at periodic checkpoints, which you can then test in the engine!
