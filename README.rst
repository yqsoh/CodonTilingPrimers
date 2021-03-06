======================================
Tiling primers for codon mutagenesis
======================================

This repository contains information and a Python script for designing codon-mutant libraries of genes using a protocol from the `Bloom lab`_.

Codon mutagenesis
-----------------
Codon mutagenesis can be used to create mutant libraries of a gene with all possible codon mutations.
Such libraries are useful for experiments such as `deep mutational scanning <https://www.ncbi.nlm.nih.gov/pubmed/25075907>`_ or `mutational antigenic profiling <http://journals.plos.org/plospathogens/article?id=10.1371/journal.ppat.1006271>`_.

The script described below can design primers for a codon mutagenesis protocol used in the Bloom lab.
This protocol randomly mutates each codon to all other possible codons.

The protocol was originally described in `Bloom (2014) <https://doi.org/10.1093/molbev/msu173>`_, and the methods section of that paper goes over the protocol in quite a bit of detail. Additionally, `here are Jesse's lab notes <JesseLabNotes.pdf>`_ for the codon mutagenesis experiments described in that paper. `Here is a schematic from Hugh Haddox <HughSchematic.pdf>`_ that graphically illustrates the process from `his paper on making libraries of HIV Env <https://doi.org/10.1371/journal.ppat.1006114>`_.

The protocol was subsequently improved by Adam Dingens by making the primers have roughly equal melting temperatures rather than equal lengths.
We think that Adam's modification to use more equal melting temperatures improves the protocol by making the rate of mutation more uniform across the gene.
These improvements are described in `Dingens et al (2017) <http://dx.doi.org/10.1016/j.chom.2017.05.003>`_ , and the script included in this repository (and described below) uses this improvement.

A few important considerations regarding our codon mutagenesis protocol:

    - The number of PCR cycles and quantifying the template concentrations is important. Don't deviate from the protocol on those unless you have a good reason.

    - It is important to use **linear PCR product** as your initial template rather than plasmid. This is because PCR is more efficient from linear templates, and the cycle numbers are optimized for that.

    - The total number of mutations that you will get out depends on how many rounds of the whole process that you perform. In the original `Bloom (2014) <https://doi.org/10.1093/molbev/msu173>`_ paper, we used three overall rounds of the process. This gave an average of almost three codon mutations per gene. That number is probably too high for most applications, so in more recent work we have used two (or sometimes even just one) round of the overall process to get an average of closer to 1 to 1.5 codon mutations per gene. (Note that recently Shirleen in the lab has found that using just one round of the whole process but increasing the number of cycles in the fragment PCR from 7 to 10 also works well in terms of giving 1 to 1.5 codon mutations, although these libraries are not yet verified by deep sequencing.)

    - We recommend that you Sanger sequence some clones to make sure things look reasonable before proceeding. An example of the types of features that you want to look at are in *S1 Fig* of `Haddox et al (2016) <https://doi.org/10.1371/journal.ppat.1006114>`_. A script that can generate these figures from Sanger sequencing data is at https://github.com/jbloomlab/SangerMutantLibraryAnalysis

Finally, note that the gene variants generated by our protocol will have a roughly **Poisson distribution** of the number of mutations per variant.
This means that some variants will have single mutations, while some will have zero or multiple mutations.
If the mean number of mutations is one, then about 37% will have no mutations, 37% will have a single mutation, and the rest will have more than one mutations (mostly two mutations).

Example uses of this protocol
+++++++++++++++++++++++++++++++

Here are some of the genes for which we have used this protocol:

    - `NP of the A/Aichi/2/1968 strain of influenza <http://mbe.oxfordjournals.org/content/31/8/1956>`_

    - `NP of A/Puerto Rico/8/1934 strain of influenza <https://dx.doi.org/10.1093/molbev/msv167>`_

    - `HA of the A/WSN/1933 strain of influenza <http://dx.doi.org/10.7554/eLife.03300>`_

    - `again for the HA of the A/WSN/1933 strain of influenza <http://www.mdpi.com/1999-4915/8/6/155>`_

    - `Env from the LAI strain of HIV <http://dx.doi.org/10.1371/journal.ppat.1006114>`_

    - `Env from the BF520 strain of HIV <http://dx.doi.org/10.1016/j.chom.2017.05.003>`_

    - `Env from the BG505 strain of HIV <https://doi.org/10.7554/eLife.34420>`_

    - `HA from the A/Perth/2009 strain of influenza <https://doi.org/10.1101/298364>`_

Alternate protocols
+++++++++++++++++++++++++++++++
There are alternative codon mutagenesis protocols that are intended to give just single codon mutations.
Here are some of these alternative protocols:

    - `Firnberg and Ostermeier (2012) <https://doi.org/10.1371/journal.pone.0052031>`_: Note that this was also the first codon mutagenesis protocol (to our knowledge)

    - `Jain and Varadarajan (2014) <https://doi.org/10.1016/j.ab.2013.12.002>`_: Note this protocol has been used with substantial success by other groups, including `papers from the Fowler lab <https://doi.org/10.1101/211011>`_

    - `Kitzman et al (2015) <http://www.nature.com/nmeth/journal/v12/n3/abs/nmeth.3223.html>`_

    - `Wrenbeck et al (2016) <http://www.nature.com/nmeth/journal/v13/n11/full/nmeth.4029.html>`_

Our lab has **not** tested any of these other protocols, so we cannot personally offer advice on how well any of them work.


Running the script to design primers
-------------------------------------

The `create_primers.py <create_primers.py>`_ Python script can be used to create ``NNN`` primers that tile the codons of a gene in both the forward and reverse direction. You can use this primers to make codon mutagenesis libraries. You might want to order these primers in the form of an `IDT 96-well plate`_.

The script takes command line arguments; for a listing of how to provide the arguments, type the following to get the help message::

    python create_primers.py -h

For instance, the file `EN72-HA_primers.txt <EN72-HA_primers.txt>`_ included in the repository can be generated with::

    python create_primers.py EN72-HA.txt EN72 1 En72-HA_primers.txt

Note that the parent gene file (`EN72-HA.txt <EN72-HA.txt>`_ in this case) should have upper-case letters for the coding sequence to mutaten and lower case letters for the flanking region.

There are a variety of optional parameters specifing primer length and melting temperature constraints; the default values for these optional parameters are displayed when you run the program with the ``-h`` option to get the help message.

You can also adjust the optional parameters described in the help message, such as::
	
    python create_primers.py EN72-HA.txt EN72 2 EN72-HA_primers.txt --minprimertm 65 --maxprimertm 66

The script works as follows:

    1) For each codon, it first makes an ORIGINAL primer of the length specified by ``--startprimerlength``

    2) If the original primer has a melting temperature (Tm) greater than the value specified by ``--maxprimertm``, then nucleotides are trimmed off one by one (first from the 5' end, then the 3' end, then the 5' end again, etc) until the melting temperature is less than ``--maxprimertm`` or the length is reduced to ``--minlength``.

    3) If the original primer has a Tm greater than ``--minprimertm``, then nucleotides are added one-by-one (first to the 3' end, then the 5' end, then the 3' end again, etc) until the melting temperature is greater than ``--minprimertm`` or the length reaches ``--maxlength``.

    4) Note that because the primers are constrained to be between ``--minprimerlength`` and ``--maxprimerlength``, the Tm may not always fall between ``--minprimertm`` and ``--maxprimertm``. This can also happen if a primer initially exceeds ``--maxprimertm`` but the first trimming that drops it below this value also drops it below ``--minprimertm``, or vice-versa if the primer is being extended to increase its melting temperature.

The  *Tm_NN* command of the `MeltingTemp module of Biopython <http://biopython.org/DIST/docs/api/Bio.SeqUtils.MeltingTemp-module.html>`_ is used to calculate Tm of primers. 
This calculation is based on nearest neighbor thermodynamics; nucleotides labeled ``N`` are given average values in the Tm calculation. 

The result of running this script is the file specified by ``outfile``. It lists the primers. All of the forward primers are have names which are the prefix specified by ``primerprefix``, then ``-for-mut``, then the codon number starting with ``firstcodon``. The reverse primers are named similarly, but with the ``for`` replaced by ``rev``. The forward primers are grouped in sets of 96 (for ordering in 96-well plates), as are the reverse primers.
The file `EN72-HA_primers.txt <EN72-HA_primers.txt>`_ shows an example output file.



.. _`Bloom lab`: http://research.fhcrc.org/bloom/en.html
.. _`IDT 96-well plate`: http://www.idtdna.com/pages/products/dna-rna/96-and-384-well-plates
