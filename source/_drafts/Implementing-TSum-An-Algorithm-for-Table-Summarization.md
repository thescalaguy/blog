---
title: 'Implementing TSum: An Algorithm for Table Summarization'
tags:
  - algorithms
  - machine learning
---

So, I am taking a break from my JVM JIT series to write this post about a table summarization algorithm that I had implemented way back in my college days. Simply put, the algorithm finds the most descriptive patterns in the table that succinctly convey the meaning of the rows contained i.e. summarize it. This post will focus on what the algorithm is and how it's implemented in JavaScript. Take this post with a grain of salt because the implementation was a part of my college assignment and hasn't been thoroughly checked for correctness.  

### What is TSum?  
TSum is the [algorithm published by Google Research](https://ai.google/research/pubs/pub41683) <sup>[[link to the pdf]](https://static.googleusercontent.com/media/research.google.com/en//pubs/archive/41683.pdf)</sup>. To quote the abstract:

> Given a table where rows correspond to records and columns correspond to attributes, we want to find a small number of patterns that succinctly summarize the dataset. ... TSum, a method that provides a sequence of patterns ordered by their "representativeness." It can decide both which these patterns are, as well as how many are necessary to properly summarize the data.  

An algorithm like this is useful in situations where the volume of data is so large that it'd be impossible for a human to deduce anything from it simply by looking at the data. For example, given patient records, a description like "most patients are middle-aged men with cholestrol, followed by child patients with chickenpox" provides a very human-readable summary of the data.  

The algorithm views finding the best pattern as a compression problem — the best pattern will give the best compression.  

### Definitions
#### Pattern and Pattern Size
A pattern $P$ is a tuple $(A_1 = u_1, ..., A_D = u_D)$ where $u_i$ can can be either a specific value or a "don't care" value represented by $\ast$. The number of matching attributes in a pattern is called the size of the pattern and is denoted by $size(P)$. An attribute is considered matching if and only if its value doesn't equal "don't care" value.  

#### Pattern List and Pattern Set
A pattern list is an ordered sequence of patterns $\mathcal{P} = [P_1, P_2 ...]$ while a pattern set is a set of patterns $\mathcal{P} = \\{P_1, P_2 ... \\}$ with no order.  

#### Compression Saving  
Compression saving of a pattern $P$ on a table $\mathcal{T}$, denoted as $Saving(P,\mathcal{T})$ is the amount of compression it can achieve. Specifically, it is the difference of bits between he uncompressed and compressed representations of the data records covered by the pattern $P$. Let $N$ be the number of records covered by pattern $P$ and $D$ be the number of attributes in $\mathcal{T}$. Then, 

$$Saving(P, \mathcal{T}) = Benefit(P, \mathcal{T}) - Overhead(P, \mathcal{T})$$  

where

$$Benefit(P, \mathcal{T}) = (N - 1) \ast \sum_{i, A_i \in matched(P)} W_i$$  

and

$$Overhead(P, \mathcal{T}) = D + log^*(N)$$


$W_i$ is the average number of bits in the i<sup>th</sup> attribute. 
$D$ is the number of attributes. 
$log^*(N) = 2 \ast \lceil log_2(N+2) \rceil$ 

#### Residue Data Table  
Given a table $\mathcal{T}$ and a pattern collection $\mathcal{P}$, the residue data table contains the records that are not covered by any of the patterns in $\mathcal{P}$.  

#### Pattern Marhsalling
Given a set of patterns $\mathcal{S}$, the pattern marshalling algorithm picks a pattern from $\mathcal{S}$ which has the highest compression saving. After every iteration, the records in the table which have been covered by the patterns chosen so far will be removed from consideration.

### Generating Patterns - Local Expansion  
Local expansion is a pattern generation strategy that tries to "grow" the patterns to increase the compression savings on the data record. In this approach, the algorithm will start with single attributes first and and find a single-condition pattern $P = \\{ (A=a) \\}$ that has the best compression saving, and then expand the pattern by adding other conditions until the compression cost cannot be improved. To find the next pattern, the same procedure is repeated, but only on the residue data table - the part of the table not covered by any of the patterns found so far.  

### Code  

We'll start with simpler code first. Let's start by writing a function to calculate the benefit.  

{% codeblock lang:javascript benefit.js %}
"use strict";
const _ = require("lodash");
/**
 * The benefit of using this pattern to represent the table
 * @param T the table containing rows
 * @param pattern the pattern to be applied
 */
function benefit(pattern, T){
    if(pattern == null) return 0;

    // a function for internal use.
    // find out the bits needed to encode the value
    function bitsFor(value) {
        if( typeof(value) === "string" || value instanceof String){
            // JS uses UTF-16
            return value.length * 16;
        }
        if( typeof(value) === "number" || value instanceof Number){
            // 64-bit for each number
            return 64;
        }
        return 0;
    }

    let N = _.filter(T, pattern).length;
    let W = 0;
    let attributes = Object.keys(pattern);

    attributes.forEach(attribute => {
        W += bitsFor(pattern[attribute]);
    });

    return (N-1) * W;
}
module.exports = benefit;
{% endcodeblock %}

The actual translation of formula to code happens from line #25. We start by finding `N` which is the number of records in the table that match the pattern. Im using [`lodash`'s `filter` function](https://lodash.com/docs#filter) to avoid the boilerplate of having to find the matching records myself. `W` is the accumulator in which I will sum the number of bits each attribute in the pattern take. `attributes` are the attributes in the pattern. Then for each attribute, we use `bitsFor` and add it up with the value of `W`. Finally, we return the value according to the formula.  

Simply put, benefit is $N - 1$ times the total number of bits each attribute in the pattern would take.

Next, let's write code to find $log^\ast(N)$

{% codeblock lang:javascript log.js %}
"use strict";
/**
 * Calculates log*(N).
 * @param N number of rows in the table
 * @returns {number}
 */
function log(N){
    return 2 * Math.log2( N+2 );
}
module.exports = log;
{% endcodeblock %}

This is a fairly straightforward translation of the formula to code and needs no explanation.  

We now have all the code we need to calculate overhead so let's do that.  

{% codeblock lang:javascript overhead.js %}
"use strict";
const _ = require("lodash");
const log = require("./log");
/**
 * The overhead of using this pattern to represent this table
 * @param T the table containing rows
 * @param pattern the pattern to be applied
 */
function overhead(pattern, T){
    let N = _.filter(T, pattern).length;
    let D = Object.keys(T[0]).length; // number of attributes
    return D + log(N);
}
module.exports = overhead;
{% endcodeblock %}

Notice that I start by `require`-ing the `log.js` module we just saw. I am using `filter` to find the number of rows in the table that match the pattern. On the next line I find the number of attributes in a given record of the table. Since the algorithm assumes no null / empty values for any attribute, we can safely pick up the 0<sup>th</sup> record and see how many attributes it contains. 

Now that we have `benefit` and `overhead` in place, let's calculate `saving`

{% codeblock lang:javascript saving.js %}
"use strict";
const benefit = require("./benefit");
const overhead = require("./overhead");
/**
 * The compression saving of a pattern P on a data table T,
 * denoted by saving(P,T) is the amount of compression it can achieve.
 * @param T the table containing rows
 * @param pattern the pattern to be applied
 */
function saving(pattern, T){
    return benefit(pattern, T) - overhead(pattern, T);
}
module.exports = saving;
{% endcodeblock %}

Now let's write code for pattern marshalling. 

{% codeblock lang:javascript pattern_marshalling.js %}
"use strict";
const _ = require("lodash");
const saving = require("./saving");
const coverage = require("./coverage");

function patternMarshalling(S, T){
    // chosen patterns
    let patterns = []; // empty = don't care for every attribute
    let remainingPatterns = _.cloneDeep(S);

    while( remainingPatterns.length > 0 ){
        // select the pattern with the top incremental compression saving
        let bTop = Number.NEGATIVE_INFINITY;
        let pTop;
        let residualTable = residue(patterns, T);

        remainingPatterns.forEach(pattern => {
            let compression = saving(pattern, residualTable);
            if( compression > bTop ) {
                bTop = compression;
                pTop = pattern;
            }
        });

        if( bTop > 0 ){
            patterns.push({
                "pattern" : pTop,
                "saving" : bTop,
                "coverage" : coverage(pTop,T)
            });
        }

        remainingPatterns = _.difference(remainingPatterns, _.filter(remainingPatterns, pTop));
    }

    return patterns;
}

function residue(patterns, T){
    patterns.forEach(pattern => {
        T = _.difference(T, _.filter(T, pattern.pattern));
    });
    return T;
}

module.exports = patternMarshalling;
{% endcodeblock %}

Given a set of patterns $\mathcal{S}$ and a table $\mathcal{T}$, we start by making a copy of the patterns passed to us. The aim is to select a pattern which gives us non-negative compression. If we find such a pattern, we add it to our `patterns` list and remove the chosen pattern from the current list of patterns. We continue the next itreation on the residue table. We repeat this until we have no more patterns to consider.  

The next piece of the puzzle is to write code for local expansion.  

{% codeblock lang:javascript local_expansion.js %}
"use strict";
const _ = require("lodash");
const expand = require("./expand");

/**
 * Local expansion pattern generation strategy that directly looks
 * for patterns that could minimize the compression cost
 * @param T an array of JSON objects
 * @returns {Array} of patterns that best summarize the data
 */
function localExpansion(T){
    let patterns = []; // final list of patterns to return

    // while we still have rows
    while( T.length > 0 ){
        // expand from an empty pattern (Algorithm 4)
        let pattern = expand(T, {});
        // stop if we cannot achieve more compression saving
        if( _.isEqual(pattern, {}) ){
            break;
        }
        // found a new pattern
        patterns.push( pattern );
        // remove all the rows that match the pattern i.e.residual table for the pattern
        T = _.difference(T, _.filter(T, pattern));
    }

    return patterns;
}

module.exports = localExpansion;
{% endcodeblock %}

As mentioned in the definitions, local expansion grows a pattern, starting with an empty pattern. The expansion is done in `expand` function.  

{% codeblock lang:javascript expand.js %}
const _ = require("lodash");
const saving = require("./saving");

/**
 * Expands a pattern to improve compression saving. See algorithm 4.
 * @param T
 * @param pattern
 * @returns {*}
 */
function expand(T, pattern){
    // find the attributes not included in the pattern
    let allAttributes = Object.keys(T[0]);
    let matchedAttributes = Object.keys(pattern);
    let attributes = _.difference(allAttributes, matchedAttributes);

    let bestPattern;
    let highestCompressionSaving = 0;

    attributes.forEach( attribute => {
        // find all the unique values for the current attribute
        let values = _.map(T, attribute);
        values = _.uniq(values);

        values.forEach( value => {
            // an expanded pattern created by appending the current
            // attribute and its value to the existing pattern
            let newPattern = _.cloneDeep(pattern);
            newPattern[attribute] = value;

            let compressionSaving = saving(newPattern, T);

            if(compressionSaving > highestCompressionSaving){
                highestCompressionSaving = compressionSaving;
                bestPattern = newPattern;
            }
        });
    });

    if( saving(bestPattern, T) > saving(pattern, T) ){
        return expand(T, bestPattern);
    }else{
        return pattern;
    }
}

module.exports = expand;
{% endcodeblock %}  

The final piece of code we need to look at is to calculate the coverage i.e. how much of the data is described by a given pattern.  

{% codeblock lang:javascript coverage.js%}
"use strict";

const _ = require("lodash");
function coverage(pattern,T){
    let matchingRows = _.filter(T,pattern);
    let coverage = matchingRows.length / T.length;
    return coverage * 100; // % of coverage
}

module.exports = coverage;
{% endcodeblock %}

Now that we have all the machinery in place, let's write a simple test. We'll take the patients example given in the paper and turn it into JSON objects. Then we'll write a simple script to run our code using this data and check whether the results make sense.  

Here's the data:

{% codeblock lang:json patients.json %}
[
  {"gender":"M","age":"adult","blood_pressure":"normal"},
  {"gender":"M","age":"adult","blood_pressure":"low"},
  {"gender":"M","age":"adult","blood_pressure":"normal"},
  {"gender":"M","age":"adult","blood_pressure":"high"},
  {"gender":"M","age":"adult","blood_pressure":"low"},

  {"gender":"F","age":"child","blood_pressure":"low"},
  {"gender":"M","age":"child","blood_pressure":"low"},
  {"gender":"F","age":"child","blood_pressure":"low"},

  {"gender":"M","age":"teen","blood_pressure":"high"},
  {"gender":"F","age":"child","blood_pressure":"normal"}
]
{% endcodeblock %} 

Here's the test:  

{% codeblock lang:javascript test1.js %}
"use strict";

const tsum = require("../index");
const table = require("./data/patients.json");
const _ = require("lodash");

let patterns = tsum.localExpansion( table );
let sorted = tsum.patternMarshalling(patterns,table);

patterns = _.shuffle(patterns);
console.log( sorted );
{% endcodeblock %} 

Now let's run this:  

{% codeblock lang:bash%}
$ node test/test1.js
[ { pattern: { age: 'adult', gender: 'M' },
    saving: 375.3852901558848,
    coverage: 50 },
  { pattern: { age: 'child', blood_pressure: 'low' },
    saving: 248.35614381022526,
    coverage: 30 } ]
{% endcodeblock %} 

The output says that the most descriptive pattern is "adult male" which makes up 50% of the rows followed by "children with low blood pressure" which make up 30% of the rows. if we look at the sample data, out of the 10 rows, 5 are "adult male" and 3 are "children with low blood pressure". So the output of the algorithm checks out.  

Finito. 
