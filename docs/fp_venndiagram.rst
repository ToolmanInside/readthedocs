###############
FPs Venn Diagram
###############

The Venn diagram of FPs of our empirical study are shown in the following figures. Each figure contains FPs which are included in one PPT. We also show other FPs whose reason are not summarized into PPTs (i.e. the reason of implementation issues of tools) in last figure. Obviously, `Oyente` shares little with `Slither` and `Securify` on PPT1, PPT3 and PPT5. In our view, this is due to `Slither` has similar detection rules with `Securify`.

**Venn of FPs due to unsatisfactory support of PPT1 (Access Control Before Payment):**

This part of FPs are due to the inconsideration of identity check. Usually these cases have conditions before payment operations, and they checks whether the invoker (i.e. the msg.sender) satisfies the condition. For examples please see our paper at section 4.2.

.. image:: /fp_venn/PPT1.jpg
    :width: 500px
    :alt: PPT1 Venn
    :align: center

**Venn of FPs due to unsatisfactory support of PPT2 (Hard-coding Payment Address):**

These FPs are due to the inconsideration of hard-coding transfer address. A hard-coding address keeps itself being exploited for malicious purposes, thus it restricts external malicious access. We show examples at section 4.3.

.. image:: /fp_venn/PPT2.jpg
    :width: 500px
    :alt: PPT2 Venn
    :align: center

**Venn of FPs due to unsatisfactory support of PPT3 (Protection by Self-defined Modifiers):**

These FPs are due to the inconsideration of function modifiers. These code blocks are not token into account in other tools. They are often used to verify the identity of invoker and restricts the transaction can only be done by certain user.

.. image:: /fp_venn/PPT3.jpg
    :width: 500px
    :alt: PPT3 Venn
    :align: center

**Venn of FPs due to unsatisfactory support of PPT4 (Protection by Execution Locks):**

These FPs are due to the inconsideration of conditions in execution paths. This PPT is to prevent the recursive entrance of the function -- elimilating the issue from root. Details are given in the section 4.4.

.. image:: /fp_venn/PPT4.jpg
    :width: 500px
    :alt: PPT4 Venn
    :align: center

**Venn of FPs due to unsatisfactory support of PPT5 (Internal Updating Before Payment):**

PPT5 is required to finish all internal work (i.e., state and balance changes) and then call the external payment function. 

.. image:: /fp_venn/PPT5.jpg
    :width: 500px
    :alt: PPT5 Venn
    :align: center

**Venn of FPs due to other reasons:**

These part of FPs are not summarized into PPTs. This is because the reasons of these FPs are implementations issues, and we think these have nothing to do with our PPTs. However, in order to explain our data explicitly, we put figures in the following.

.. image:: /fp_venn/other.jpg
    :width: 500px
    :alt: other Venn
    :align: center