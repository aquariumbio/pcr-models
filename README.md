# Aquarium PCR Models

This set of libraries can be used to model reaction compositions and thermocycler programs for PCR. 

Reaction compositions are modeled by `class PCRComposition`, which includes descriptions of the composition like this one:

```ruby
'qPCR1' => {
  polymerase: {
    input_name: POLYMERASE,
    qty: 16, units: MICROLITERS,
    sample_name: 'Kapa HF Master Mix',
    object_name: 'Enzyme Stock'
  },
  forward_primer: {
    input_name: FORWARD_PRIMER,
    qty: 0.16,  units: MICROLITERS
  },
  reverse_primer: {
    input_name: REVERSE_PRIMER,
    qty: 0.16,  units: MICROLITERS
  },
  dye: {
    input_name: DYE,
    qty: 1.6, units: MICROLITERS,
    sample_name: 'Eva Green',
    object_name: 'Screw Cap Tube'
  },
  water: {
    input_name: WATER,
    qty: 6.58, units: MICROLITERS
  },
  template: {
    input_name: TEMPLATE,
    qty: 7.5, units: MICROLITERS
  }
}
```

You can create a `PCRComposition` object with this composition using a factory method:

```ruby
composition = PCRCompositionFactory.build(
  program_name: "qPCR1"
)
```

Individual components are modeled by the `ReactionComponent` class, created during initialiization of `PCRComposition`.

Reaction programs are modeled by `class PCRProgram`, which includes descriptions of the program like this one:

```ruby
'qPCR1' => {
  program_template_name: 'NGS_qPCR1',
  layout_template_name: 'NGS_qPCR1',
  volume: 32,
  steps: {
    step1: {
      temperature: { qty: 95, units: DEGREES_C },
      duration: { qty: 3, units: MINUTES }
    },
    step2: {
      temperature: { qty: 98, units: DEGREES_C },
      duration: { qty: 15, units: SECONDS }
    },
    step3: {
      temperature: { qty: 62, units: DEGREES_C },
      duration: { qty: 30, units: SECONDS }
    },
    step4: {
      temperature: { qty: 72, units: DEGREES_C },
      duration: { qty: 30, units: SECONDS }
    },
    step5: { goto: 2, times: 34 },
    step6: {
      temperature: { qty: 12, units: DEGREES_C },
      duration: { qty: 'forever', units: '' }
    }
  }
}
```

Like `PCRComposition`, you can create a `PCRProgram` object with this program using a factory method:

```ruby
program = PCRProgramFactory.build(
  program_name: "qPCR1", 
  volume: composition.volume
)
```

`PCRProgram` can automatically generate tables for `show` blocks:

```ruby
show do
  title "Program Setup"
    
  note "Enter these settings into the thermocycler:"
  table program.table
end
```

the `table` renders as:

<table><tr><td style="border: 1px solid gray; text-align: center">Step</td><td style="border: 1px solid gray; text-align: center">Temperature</td><td style="border: 1px solid gray; text-align: center">Duration</td></tr><tr><td style="border: 1px solid gray; text-align: center">step1</td><td style="border: 1px solid gray; text-align: center">95 &deg;C</td><td style="border: 1px solid gray; text-align: center">3 min</td></tr><tr><td style="border: 1px solid gray; text-align: center">step2</td><td style="border: 1px solid gray; text-align: center">98 &deg;C</td><td style="border: 1px solid gray; text-align: center">15 sec</td></tr><tr><td style="border: 1px solid gray; text-align: center">step3</td><td style="border: 1px solid gray; text-align: center">62 &deg;C</td><td style="border: 1px solid gray; text-align: center">30 sec</td></tr><tr><td style="border: 1px solid gray; text-align: center">step4</td><td style="border: 1px solid gray; text-align: center">72 &deg;C</td><td style="border: 1px solid gray; text-align: center">30 sec</td></tr><tr><td style="border: 1px solid gray; text-align: center">step5</td><td style="border: 1px solid gray; text-align: center">goto step 2</td><td style="border: 1px solid gray; text-align: center">34 times</td></tr><tr><td style="border: 1px solid gray; text-align: center">step6</td><td style="border: 1px solid gray; text-align: center">12 &deg;C</td><td style="border: 1px solid gray; text-align: center">forever </td></tr></table>

Master mixes are handled by `module MasterMixHelper`. This module contains, among other things, a method for grouping `Operations` by shared inputs, and a `show` method for making a master mix based on a `PCRComposition` object.

All three make use of the `Units` library (part of [Standard Libraries](https://github.com/klavinslab/standard-libraries), which, for example, renders

```ruby
volume = {qty: 6.58,  units: MICROLITERS}
Units.qty_display(volume)
```

as 6.58 µl. This is especially useful for units with greek letters, which are often lost when they are hard-coded in protocols.
