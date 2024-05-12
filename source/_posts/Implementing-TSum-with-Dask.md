---
title: Implementing TSum with Dask
tags:
  - algorithms
date: 2024-04-07 11:45:43
---


In one of the [previous blog posts](/2018/10/21/Implementing-TSum-An-Algorithm-for-Table-Summarization/) I'd written about implementing TSum, a table-summarization algorithm from Google Research. The implementation was written using Javascript and was meant for small datasets that can be summarized within the browser itself. I recently ported the implementation to [Dask](https://www.dask.org/) so that it can be used for larger datasets that consist of many rows. In a nutshell, it lets us summarize a Dask DataFrame and find representative patterns within it. In this post we'll see how to use the algorithm to summarize a Dask DataFrame, and run benchmarks to see its performance. 

## Before We Begin  

Although the library is designed to be used in production on data stored in a warehouse, it can also be used to summarize CSV or Parquet files. In essence, anything that can be read into a Dask DataFrame can be summarized.

## Getting Started

### Summarizing data

Imagine that we have customer data stored in a datawarehouse that we'd like to summarize. For example, how would we best describe the customer's behavior given the data? In essence, we'd like to find patterns within this dataset. In scenarios like these, TSum works well. As an example of data summarization, we'll use the patient data given in the [research paper](https://static.googleusercontent.com/media/research.google.com/en//pubs/archive/41683.pdf) and pass it to the summarization algorithm.  

We'll begin by adding a function to generate some test data. 

{% code lang:python %}
def data(n=1):
    return [
        {"gender": "M", "age": "adult", "blood_pressure": "normal"},
        {"gender": "M", "age": "adult", "blood_pressure": "low"},
        {"gender": "M", "age": "adult", "blood_pressure": "normal"},
        {"gender": "M", "age": "adult", "blood_pressure": "high"},
        {"gender": "M", "age": "adult", "blood_pressure": "low"},
        {"gender": "F", "age": "child", "blood_pressure": "low"},
        {"gender": "M", "age": "child", "blood_pressure": "low"},
        {"gender": "F", "age": "child", "blood_pressure": "low"},
        {"gender": "M", "age": "teen", "blood_pressure": "high"},
        {"gender": "F", "age": "child", "blood_pressure": "normal"},
    ] * int(n)
{% endcode %}

We'll then add code to summarize this data.  

{% code lang:python %}
import json
import time

import cattrs
import dask.dataframe as dd
import pandas as pd
import tabulate

from tsum import summarize

if __name__ == "__main__":
    from dask.distributed import LocalCluster

    cluster = LocalCluster(n_workers=1, nthreads=8, diagnostics_port=8787)
    client = cluster.get_client()

    df = pd.DataFrame.from_records(data=data(n=1))
    ddf = dd.from_pandas(df, npartitions=4)
    t0 = time.perf_counter()
    patterns = summarize(ddf=ddf)
    t1 = time.perf_counter()

    dicts = [cattrs.unstructure(_) for _ in patterns]
    print(json.dumps(dicts, indent=4))
{% endcode %}

Upon running the script we get the following patterns.

{% code lang:json %}
[
    {
        "pattern": {
            "gender": "M",
            "age": "adult"
        },
        "saving": 3313,
        "coverage": 50.0
    },
    {
        "pattern": {
            "age": "child",
            "blood_pressure": "low"
        },
        "saving": 1684,
        "coverage": 30.0
    }
]
{% endcode %}

This indicates that the patterns that best describe our data are "adult males", which comprise 50% of the data, followed by "children with low blood pressure", which comprise 30% of the data. We can verify this by looking at the data returned from the `data` function, and from the patterns mentioned in the paper.

### Running benchmarks

To run the benchmarks, we'll modify the script and create DataFrames with increasing number of rows. The benchmarks are being run on my local machine which has an Intel i7-8750H, and 16GB of RAM. The script which runs the benchmark is given below.

{% code lang:python %}
if __name__ == "__main__":
    from dask.distributed import LocalCluster

    cluster = LocalCluster(n_workers=1, nthreads=8, diagnostics_port=8787)
    client = cluster.get_client()
    table = []

    for n in [1, 1e1, 1e2, 1e3, 1e4, 1e5, 1e6]:
        df = pd.DataFrame.from_records(data=data(n=n))
        ddf = dd.from_pandas(df, npartitions=4)
        t0 = time.perf_counter()
        summarize(ddf=ddf)
        t1 = time.perf_counter()
        table.append(
            {
                "Rows": len(ddf),
                "Time Taken (seconds)": (t1 - t0),
            }
        )

    print(tabulate.tabulate(table))
{% endcode %}   

This is the output generated. As we can see, it takes 17 minutes for 1e6 rows.

{% code %}
--------  ---------
      10    14.5076
     100    24.1455
    1000    23.4862
   10000    23.4842
  100000    32.8378
 1000000   121.013
10000000  1050.46
--------  ---------
{% endcode %}   

## Conclusion  

That's it. That's how we can summarize a Dask DataFrame using TSum. The library is available on PyPI and can be installed with the following command.  

{% code lang:bash %}
pip install tsum
{% endcode %}  

[The code is available on GitHub.](https://github.com/thescalaguy/tsum-dask) Contributions welcome.