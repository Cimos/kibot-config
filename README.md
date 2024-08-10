# kibot-config

Kibot config is meant for building kicad pcb datapacks using github actions. While I have chosen a flexible license to for this project to allow the community to do what they want. I encourage anyone using this to make your project opensource.

## Use case

I have indented for this repository to make building kicad pcb artifacts as easy as possible.

### Step 1: Add Github Action

Add `main.yaml` to `.github/workflows/` on the root directory of the github repository.

### Step 2: Add options.yaml

Add `options.yaml` to the root directory of the github repository.

### Step 3: Commit changes

Commit changes.

### Example 

[Mad_RP2040](https://github.com/Cimos/Mad_RP2040/) is an example of how kibot-config can be used. 

### Example project folder structure

Your project folder structure will look something like the following.

```text
SomeProject/
  .github/
  SomeProject.kicad_pro
  SomeProject.kicad_sch
  SomeProject.kicad_pcb
  options.yaml

  .github/
    workflows/

  workflows/
    main.yaml
```

## Future work

* Add support for variants
* Add 2d render generation
* Add GIF generation
* Add github action which calls a default action with the default `options.yaml` in this repository.
