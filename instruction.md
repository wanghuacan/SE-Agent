# SWEAgent Usage Instructions

## Environment Setup
This section covers the environment requirements for SWEAgent itself. Our code is mainly organized in the SE directory.
### Source Installation

1. Install dependencies:
```bash
pip install --editable .
```

2. Verify installation:
```bash
sweagent --help
```


## Basic Run Commands

### Running Examples

Basic run command:
```bash
python SE/basic_run.py --config /home/uaih3k9x/630_swe/SE/configs/se_configs/dpsk29.yaml
```

Run command using crossover operator:
```bash
python SE/basic_run.py --config /home/uaih3k9x/SE-Agent/SE/configs/se_configs/crossover_test.yaml
```

### Command Parameter Description

- `--config`: Specify configuration file path, allows customization of different configuration combinations
- For other parameters, please refer to specific configuration file settings

### Strategy Parameter Details

The `strategy` parameter defines the iteration flow of the entire execution strategy, using a multi-round processing approach:

```yaml
strategy:
  iterations:
    - base_config: "SE/configs/base_configs/baseconfig1.yaml"
      operator: null
    - base_config: "SE/configs/base_configs/baseconfig2.yaml"
      operator: "alternative_strategy"
    - base_config: "SE/configs/base_configs/baseconfig2.yaml"
      operator: "crossover"
```

#### Parameter Explanation:

1. **iterations**: Defines the execution sequence of 3 iteration steps
   - Each iteration represents a processing phase
   - Executed sequentially according to array order

2. **base_config**: Specifies the base configuration file used for each iteration
   - `baseconfig1.yaml`: Base configuration used in the first round
   - `baseconfig2.yaml`: Base configuration used in subsequent rounds
   - Different base_configs may contain different model parameters, prompts, etc.

3. **operator**: Specifies the operator used for each iteration (optional)
   - `null`: No additional operator used, only base configuration
   - `"alternative_strategy"`: Uses alternative strategy operator to generate multiple solutions
   - `"crossover"`: Uses crossover operator to combine and optimize results from previous iterations

#### Execution Flow:

1. **Round 1**: Uses base configuration from baseconfig1.yaml, no additional operator loaded
2. **Round 2**: Switches to baseconfig2.yaml configuration, loads alternative_strategy operator to generate alternative strategies
3. **Round 3**: Continues using baseconfig2.yaml, loads crossover operator to perform crossover optimization on previous results

This design allows using different strategy combinations at different stages, achieving evolution and optimization of solutions through the crossover operator.

## Framework Features

### Configuration File Customization

- Can customize different configuration combinations in the config directory
- Supports flexible parameter configuration and strategy selection
- Configuration files use YAML format, easy to modify and extend

### Operator Extension

- Can continue writing custom operators in the operator directory
- Supports modular operator combinations
- Provides rich operator interfaces for functionality extension

### Extensibility

- Framework design supports multiple runtime modes
- Can add new functional modules as needed
- Supports different evaluation strategies and optimization schemes

## TODO

- [ ] Implementation of solution selection (evaluation function)
- [ ] Development of enhanceoperator operator functionality
- [ ] Complete testing and validation of se_run.py (currently use SE/basic_run.py), support checkpoint resumption