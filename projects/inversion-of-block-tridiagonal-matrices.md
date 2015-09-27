---
layout: project
title: Inversion of Block Tridiagonal Matrices
permalink: /projects/inversion-of-block-tridiagonal-matrices/
---
The bachelor thesis from 2010 I did in collaboration with Rasmus Koefoed Jakobsen. 

The abstract reads: We have implemented a prototype of the parallel block tridiagonal matrix inversion algorithm presented by Stig Skelboe [Ske09] using C# and the Microsoft.NET platform. The performance of our implementation was measured using the Mono runtime on a dual Intel Xeon E5310 (a total of eight cores) running Linux. Within the context of the software and hardware platform, we feel our results support Skelboes theories. We achieved speedups of 7.348 for inversion of block tridiagonal matrices, 7.734 for LU-factorization, 7.772 for minus--matrix--inverse-matrix multiply (-A*B^-1), and 7.742 for inversion of matrices.

- [Source code and technical report on GitHub](https://github.com/egil/Inversion-of-Block-Tridiagonal-Matrices)
- Technical report is also available from [the faculties website](http://diku.dk/research/Publikationer/tekniske_rapporter/2010/)

