Clairvoyance
============

.. image:: eye.png
    :width: 200px
    :alt: Solidity logo
    :align: center

Static program analysis still plays a key role in detecting and fixing vulnerabilities (e.g., reentrancy) in smart contracts. However, the existing static analyzers still suffer from two major limitations: 

- lack of intra-contract analysis
- lack of path feasibility due to the techniques used by programmers to prevent reentrancy (e.g.,permission controls, hard-coded addresses and execution locks). 

In this work, we present Clairvoyance, a cross-function and cross-contract static analysis by identifying infeasible paths for detecting reentrancy vulnerabilities in smart contracts. 

.. note::
    To reduce FNs, we enable, for the first time, a cross-contract call chain analysis by tracking possibly tainted paths. To reduce FPs, we have conducted extensive empirical studies and summarized five major path protective techniques (PPTs) to support fast yet precise path feasibility checking.

We have implemented our approach and compared Clairvoyance with three state-of-the-art approaches on **17770** real-worlds contracts. Results show that Clairvoyance yields the best detection accuracy among all tools and also finds **76** unknown reentrancy vulnerabilities. In addition, Clairvoyance is comparable to the fastest rule-based tool (i.e., Slither) in  analysis time, but significantly faster than verification-based tools Oyente and Securify.

In this website, we sample some vulnerable smart contract code which are pointed out by Clairvoyance and show our exploits. Each exploit consists of the metadata of contract (e.g. transaction count, ethers it involved), the exploit code and concise explanations. Exploits will be continuously updated in the future.

For additional evaluation and explanations on vulnerabilities, please see `Additional evaluations <https://toolman-demo.readthedocs.io/en/latest/evaluations.html>`_.

.. toctree::
    :maxdepth: 2

    evaluations.rst
    exploits.rst

