# neural-data-normalizer

A simple data normalizer to be used with neural networks.

As my grandfather used to say (probably), **neural networks** are dumb. When they're born and when you need to train them just to see how all the magic works, its a pain in the&hellip; neck.

This library **converts datasets of human data** into **arrays of bits** understandable by neurons (duh).

*Disclaimer*:
This script was made when I tested the awesome [synaptic.js](https://github.com/cazala/synaptic) neural network library and might not suit all sorts of inputs. It's mainly meant to be able to quickly take test data from examples given around the web for input into neural networks.

## Install

```bash
yarn add @yodasws/neural-data-normalizer
```

```bash
npm install --save @yodasws/neural-data-normalizer
```

## Example

Consider this. I'm trying to plug a neural network into my Arduino Connected Garden and I've got the following data. I want my network to know when or when not to water my plants on its own (whatever the units are for now).

```json
{ "soilhumidity": 500, "airtemp": 32, "airhum": 18, "water": true, "plants": ["tomatoes", "potatoes"] },
{ "soilhumidity": 1050, "airtemp": 40, "airhum": 21, "water": true, "plants": ["potatoes", "asparagus"] },
{ "soilhumidity": 300, "airtemp": 100, "airhum": 90, "water": false, "plants": ["asparagus", "tomatoes"] },
{ "soilhumidity": 950, "airtemp": 103, "airhum": 26, "water": true, "plants": ["asparagus", "asparagus"] },
{ "soilhumidity": 1050, "airtemp": 8, "airhum": 26, "water": true, "plants": ["tomatoes", "tomatoes"] },
{ "soilhumidity": 1050, "airtemp": 56, "airhum": 26, "water": true, "plants": ["potatoes", "french fries"] },
```

In the end, my output is "should I water the plants?": `water: true` and the rest are my inputs. Let's do this.

```javascript
const normalizer = new Normalizer(sampleData);

// setting required options and normalize the data
normalizer.setOutputProperties(['water']);
normalizer.normalize();

// find useful information about your data
// to pass to your neural network
const nbrInputs = normalizer.getInputLength();
const nbrOutputs = normalizer.getOutputLength();

const metadata = normalizer.getDatasetMetaData();
const inputs = normalizer.getBinaryInputDataset();
const outputs = normalizer.getBinaryOutputDataset();

console.log(metadata);
console.log(inputs);
console.log(outputs);
```

There you should have all useful the information to give to your network, like the **number of inputs** and **outputs**, you get **binarized dataset suitable for neural networks**, and even some *metadata*.

```javascript
{ soilhum: { type: 'number', min: 300, max: 1050, distinctValues: null },
  airtemp: { type: 'number', min: 8, max: 103, distinctValues: null },
  airhum: { type: 'number', min: 18, max: 90, distinctValues: null },
  water: { type: 'boolean', min: 0, max: 1, distinctValues: null },
  plants:
   { type: 'array',
     min: null,
     max: null,
     distinctValues: [ 'tomatoes', 'potatoes', 'asparagus', 'french fries' ] } }

[ [ 0.266667, 0.252632, 0, 1, 1, 0, 0 ],
  [ 1, 0.336842, 0.041667, 0, 1, 1, 0 ],
  [ 0, 0.968421, 1, 1, 0, 1, 0 ],
  [ 0.866667, 1, 0.111111, 0, 0, 1, 0 ],
  [ 1, 0, 0.111111, 1, 0, 0, 0 ],
  [ 1, 0.505263, 0.111111, 0, 1, 0, 1 ] ]

[ [ 1 ],
  [ 1 ],
  [ 0 ],
  [ 1 ],
  [ 1 ],
  [ 1 ] ]
```

## Why metadata?

Consider a real example where you actually start to understand what neural networks are and start implementing one. Soon you realize the biggest challenge is data formatting. When you **activate the network** with you data you realize you also need to **normalize the new data input** as well.

So you need to save your metadata information from earlier so you can reuse it! This implies training data *MUST* contain min and max values at some point.

Then on new unknown input you just have to recall the normalizer with the metadata:

```javascript
const normalizer = new Normalizer(newData);

normalizer
    .setDatasetMetaData(networkObject.metadata)
    .setOutputProperties(['water']);

const input = normalizer.getBinaryInputDataset()[0];
```

And now you can activate your neural network!

See also `test.js` for an example using [synaptic.js](https://github.com/cazala/synaptic).
