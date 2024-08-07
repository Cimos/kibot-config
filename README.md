# kibot-config

Kibot config for building a datapack for kicad using github actions.

## Github Action

You will need to add a github action with the following to the project that you want to be built.

```yaml
name: build
on:
  push:
    paths:
    - '**.kicad_sch'
    - '**.kicad_pcb'
  pull_request:
    paths:
      - '**.kicad_sch'
      - '**.kicad_pcb'
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    # Use current kicad repo
    - uses: actions/checkout@master
    
    # Run bash command to get kicad file name. 
    - name: Get Kicad Project Name
      run: echo "PROJECT_NAME=$(basename *.kicad_pro .kicad_pro)" >> $GITHUB_ENV
    # Checkout kibot config repo
    - name: Get Kibot Config
      uses: actions/checkout@master
      with:
        repository: Cimos/kibot-config
        path: ./kibot-config

    
    # Use kibot build.
    - uses: INTI-CMNB/KiBot@v1.7.0
      with:
        config: kibot-config/build.kibot.yaml
        dir: output
        schema: '${{ env.PROJECT_NAME }}.kicad_sch'
        board: '${{ env.PROJECT_NAME }}.kicad_pcb'
    
    # Update artifacts
    - name: Upload Results
      uses: actions/upload-artifact@v2
      with:
        name: output
        path: output
```
