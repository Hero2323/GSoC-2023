# GSoC-23

<p align="center">
  <img src="files/images/GSoC_logo.svg" alt="GSoC Logo">
  <img src="files/images/6004541.png" alt="Image">


<div align="center">
    <h2>
        <a href="https://summerofcode.withgoogle.com/programs/2023/projects/bAs5hkHh">
            Reducing Fossology's False Positive Copyrights
        </a> @ 
        <a href="https://www.fossology.org/">
            FOSSology
        </a>
    </h2>
</div>

<h1 align="center" id="project-details">Project Details</h1>

The aim of this project is to enhance and automate Fossology's copyright detection process. As it stands, Fossology largely depends on regex, initially employing it to pinpoint all *potential* copyright notices. Subsequently, human intervention is required to vet every copyright declaration and to weed out the inaccuracies. Given its rigorous extraction method to ensure no copyright notices are overlooked, it often demands extensive manual hours to thoroughly review the results.

This project's primary objective is to serve as an additional layer. Once Fossology's copyright agent has extracted all potential copyrights, machine learning, specifically natural language processing (NLP), will be employed to sift out the false positives.

Moreover, another aim is to tidy up copyright statements that embed extraneous information not pertaining to the copyright itself—such as some license notifications that accompany certain copyright notices. Cleaning up this clutter typically demands significant human effort.

*This project is a continuation of the GSoC-21 Project started by [Kaushlendra](https://github.com/Kaushl2208) who currently, along with other mentors, is mentoring me for this project. The previous project can be found [here](https://github.com/Kaushl2208/GSoC2021)*

<h1 align="center" id="contributions">Contributions</h1>

<h2>1. Dataset Creation for Cleared Copyrights for Model Training</h2>

The dataset, which was manually labelled and inspected, comprises numerous project repositories like Fossology, Tensorflow, Kubernetes, among others. It's accessible to all via a Google spreadsheet, with individual sheets for each dataset and a consolidated sheet merging all labeled datasets.

- [Google Sheet Link](https://docs.google.com/spreadsheets/d/132NnbJT4nqb-hxPX-XRFvUWTUg9SW0-ueW2YkpykgSk/edit?usp=sharing)

<h2>2. Development and Training of the False Positive Copyright Detection Model</h2>

This endeavor comprises two facets: text preprocessing and model training.

<h3>Preprocessing</h3>

Exploratory efforts were directed towards diverse preprocessing techniques suitable for copyright statements. Given that copyrights deviate from standard English patterns—integrating punctuations, dates, copyright symbols, and lacking the usual English sentence structure—conventional text preprocessing might not be optimal.

The most effective preprocessing incorporated the following sequential steps:
  1. Using Named Entity Recognition to mask the copyright holder's identity.
     - Before: Copyright (c) 2000, `Frank Warmerdam`
     - After: Copyright (c) 2000, `ENTITY`
     - This adjustment mitigates biases and facilitates model generalization.
  2. Substitute dates (e.g., 2003, 1999) with `DATE` to preserve the structural integrity of copyright statements.
  3. Eradicate any residual numbers.
  4. Replace copyright symbols (©, (c), (C)) with `COPYRIGHTSYMBOL` to enhance model comprehension.
  5. Convert emails to `EMAIL`.
  6. Expunge remaining special characters.
  7. Apply text lowercasing.
  8. Implement TF-IDF vectorization.
     - TF-IDF was favored after extensive benchmarking against other vectorization techniques, such as GloVe and FastText.

<h3>Model</h3>

Extensive evaluations on numerous machine learning and some deep learning techniques revealed traditional machine learning's superiority, primarily due to dataset size constraints. The finalized model selected was SVM. Comprehensive tests were run for both the vectorizer and the model to determine optimal parameters.

The project's GitHub repository detailing tests, preprocessing approaches, and decluttering is accessible [here](https://github.com/Hero2323/Fossology-Reducing-Copyrights). Furthermore, a notebook illustrating the model and associated functions in a user-friendly format is included.

<h2>3. Declutter Functionality</h2>

Decluttering aims to excise parts of the copyright statement not inherently related to the copyright.

Example:
  - Before: Copyright (C) 2000, Larry Wall. Everyone is permitted to...
  - After: Copyright (C) 2000, Larry Wall.

Given the specificity of this task, there weren't any pre-trained models to assist in the preliminary labeling. As a starting point, a new dataset was crafted for training, using `doccano` for streamlined labeling.

The essence of the NER method is the singular `COPYRIGHT` entity, which marks every portion of the copyright. Subsequent text, not recognized as `COPYRIGHT`, is deemed as clutter and is eliminated.

SpaCy, extensively utilized for detecting copyright entities, was leveraged to train a model for this objective.

<h2>4. Evaluation and Performance Metrics</h2>

Performance metrics for both the false positive detection and decluttering models are presented.

<h3>False Positive Detection Performance</h3>

The model boasts an F1-Score of 98.87% on the test dataset. It's worth noting that the model's training did not include non-English samples. Some inaccuracies arose from non-English or binary text fragments. In practical scenarios, performance exceeds 99%.

Additionally, here's a breakdown of performance on external datasets, previously unseen by the model:

1. **Tensorflow**: F1-Score of 98.62%
2. **Kubernetes**: F1-Score of 96%
3. **Feature Extraction dataset** (derived from Linux, it was created by [this paper](https://www.jstage.jst.go.jp/article/transinf/E103.D/12/E103.D_2020EDL8089/_article)): F1-Score of 99.85%
4. Subsequent tests post-integration demonstrated comparable performance metrics.

Conclusively, while there's room for enhancement, the model has notably achieved its goal, potentially conserving countless hours previously dedicated to rectifying false positives.


<h3> Decluttering Performance </h3>

Understanding the decluttering performance of the model can be somewhat challenging. While the concept of copyright statements might seem straightforward, the real-world variations in these statements introduce significant complexity. The varied ways a copyright notice might be embedded amidst other information creates many edge cases for any algorithm to consider.

For illustration, consider the following examples where the highlighted portion indicates what our model identifies as the core copyright notice:
 1. `Copyright (c) 2001 Bill Bumgarner <bbum@friday.com>` License: MIT, see below.
 2. `Copyright (C) 2001 Python Software Foundation, www.python.org Taken from     Python2.2`, License: PSF - see below.
 3. `Copyright (C) 2001 Python Software Foundation` , www.python.org `Taken from  Python2.2`, License: PSF - see below.
 4. `copyright, i.e., "`  `Copyright (c) 2001, 2002, 2003, 2004, 2005, 2006 Python Software Foundation` ; All Rights Reserved" are retained in Python alone or in any derivative version prepared by Licensee.
 5. `Copyright (C) 2007 Free Software Foundation, Inc. <https://fsf.org/>'` Everyone is permitted to copy and distribute verbatim copies of this license document, but changing it is not allowed.
 6. `Copyright 2019 Ansible Project GNU General Public License v3.0+ (see COPYING or https://www.gnu.org/licenses/gpl-3.0.txt)`
 7. `(c) 2014, James Tanner <tanner.jc@gmail.com>`
 8. `(c) 2017 Ansible Project GNU General Public License v3.0+ (see COPYING or https://www.gnu.org/licenses/gpl-3.0.txt) from __future__ import (absolute_import, division, print_function) metaclass__` = type
 9. `(c) 2013, bleader Written by bleader <bleader@ratonland.org> Based on` pkgin module written by Shaun Zinck that was based on pacman module written by Afterburn <https://github.com/afterburn> that was based on apt module written by Matthew Williams <matthew@flowrout>
 10. `(c) 2005, 2014 jQuery Foundation, Inc.` | jquery.org/license */
 11. `Copyright 2005-2008 The Android Open Source Project This product includes software developed as part of The Android Open Source Project` (http://source.android.com).
 12. `Copyright (c) 1998 Hewlett-Packard CompanydescsRGB IEC61966-2.1sRGB IEC61966-2.1XYZ óQ ÌXYZ XYZ o¢8õXYZ b·ÚXYZ $ ¶ÏdescIEC http://www.iec.chIEC`

From these examples, a pattern emerges. The model demonstrates a proficiency in identifying distinct copyright statements. However, it exhibits challenges when the statements are embedded within additional context or exhibit unique formatting. These examples, all of which are **out-of-dataset and previously unseen by the model**, highlight its current limitations. Despite being trained on a dataset of about 4000 labeled texts, the model encounters difficulties with unfamiliar structures and formats, such as the GNU license. This suggests that while the model has learned well from its training data, there's a need for broader exposure to diverse examples to enhance its generalization capabilities.


While the current accuracy of 50% on new and unique cases is a promising start, it's evident that further refinement and a larger training dataset would be beneficial.


<h2>5. Creation of the Safaa package</h2>

After the completion of the model development, the next logical step was to encapsulate it into a package. This would not only make the binaries of the model easily accessible but would also provide a structured way for users to interact with it. Thus, the `Safaa` package was born.

<!-- TODO: Change the package link -->
You can find the package on [PyPI](https://pypi.org/project/copyrightfpd/) and its source code is hosted on [GitHub](https://github.com/fossology/copyrightfpd). This package simplifies the user experience by offering two primary functions: `predict` and `declutter` as well as other functions for training purposes. For a deeper dive into how each function works, the linked documentation provides comprehensive insights.

**A copy of the package source code is provided in this Repository**

<h2>6. Writing traing and testing scripts </h2>

The dynamic nature of copyright requires our model to be continuously updated with new data. To facilitate this iterative improvement, I've developed scripts that automate the processes of training and testing. These scripts are set to be used every few months, ensuring that the model remains updated with the latest data trends.

For those interested in the intricacies of these scripts or perhaps even in contributing to their development, they are hosted alongside the package code on [this GitHub repository](https://github.com/fossology/copyrightfpd).

<h2>7. Integration with Fossology</h2>

Merging our work into the existing Fossology infrastructure was streamlined by the creation of our package. The `copyrightfpd` package abstracted many complexities, making the integration process smoother. 

A notable mention goes to [Kaushlendra](https://github.com/Kaushl2208) who significantly eased the integration into Fossology's PHP codebase, leveraging his invaluable work from the GSoC-21 project.


<h2>8. Documentation and Pull Requests</h2>

A crucial aspect of development is ensuring that all changes and additions are well-documented. This not only helps current collaborators but also future developers who might work on this project. 

1. I've created a Pull Request that integrates my package into the Fossology Codebase. You can review it [here](https://github.com/fossology/fossology/pull/2589).
2. The source code of my package has been uploaded via this Pull Request into the Fossology's maintained repository. This will be crucial for future updates. [Link](https://github.com/fossology/copyrightfpd/pull/1)
3. For those interested in the journey and iterative progress of this project, I've maintained weekly progress updates which can be explored [here](https://fossology.github.io/gsoc/docs/2023/copyrights).

**Additionally, there is a [notebook](./files/project-showcase.ipynb) provided with this project, which showcases the functionalities of the package, as well as more of the code I created and used during the package creation.**

<h1 align="center" id="deliverables">Deliverables</h1>
  
<div align="center">

| Tasks                            | Planned | Completed              |
| -------------------------------- | ------- | ----------------------- |
| Dataset Creation                 |   ✅    |   ✅                    |
| False Positive Detection Model |   ✅    |   ✅                    |
| Decluttering Model                     |    ✅     |   ✔️ (Partial, requires further improvements) |
| Safaa Package Creation            |     ❌    |   ✅  |
| Integration                      |   ✅    |   ✅                    |
| Training and Utility Scripts     |   ✅    |   ✅                    |

</div>

<br>

<h1 align="center" id="future-goals">Future Goals</h1>

<h2> Expand the Dataset </h2> 
As discussed in the decluttering performance, there's an evident need for a larger and more diverse dataset. Incorporating a variety of real-world copyright notices, particularly those not covered in the current dataset, will aid in improving model accuracy and robustness.
<h2> Language Adaptability </h2> 

The current model mainly caters to English copyright notices. Future endeavors should consider expanding its capabilities to other languages, making the tool more universally applicable.

<h2> Incorporate Feedback Loop </h2>

Implement a mechanism to capture feedback from Fossology users. This would provide valuable insights into the model's real-world performance and areas of improvement.

<h2> Optimize Model Parameters </h2>
While SVM was found to be the most suitable model for the current dataset, continuous evaluations and benchmarking should be carried out as the dataset evolves and grows.

<h2> Enhanced Decluttering </h2>
Further refine the decluttering mechanism. As the decluttering model encounters new edge cases, retraining with these instances will be paramount to its ongoing improvement. 
<br>
<br>

<h1 align="center" id="key-takeaways">Key Takeaways</h1>

Navigating this project deepened my appreciation for the intricacies of machine learning and natural language processing. With every challenge, I learned something new. Below are some of the key experiences and takeaways:

## Labeling Datasets

- **Quality Assurance**: Ensured high-quality data for better machine learning outcomes.
- **Efficient Automation**: Developed automation scripts to streamline the data labeling process.

## Named Entity Recognition (NER)

- **Dataset Creation**: Curated datasets tailored for NER tasks.
- **Precision in Labeling**: Adopted meticulous labeling procedures to accurately identify entities.

## SpaCY for NER Applications

- **Proficiency with SpaCY**: Leveraged SpaCY for advanced NER and natural language processing tasks.
- **Model Training & Deployment**: Gained hands-on experience in training and deploying models using SpaCY.

## Python Package Development and Deployment

- **Code Packaging**: Converted scripts into modular Python packages.
- **Publication to PyPI**: Successfully published packages on the Python Package Index.

## Preprocessing & Experimentation

- **Innovative Preprocessing**: Created custom a preprocessing functions for copyright notice tasks.
- **Dataset Organization**: Improved methods for dataset structuring and use.

## Model Optimization & Vectorization

- **Parameter Tuning**: Adjusted model parameters for better performance.
- **Vectorizer Exploration**: Tested various vectorization techniques to find the best fit.

## Collaborative Endeavors

- **Team Dynamics**: Engaged in regular discussions with my mentors, ensuring we were aligned in our approach and vision.
- **Work Presentation**: Presented project findings and details to a range of audiences.

## Git Mastery

- **Version Control**: Improved skills in using git for effective version control and team collaboration.

<h1 align="center" id="acknowledgements">Acknowledgements</h1>

I extend my deepest gratitude to my mentors [Gaurav Mishra](https://github.com/GMishx), [Anupam Ghosh](https://github.com/ag4ums), [Ayush Bharadwaj](https://github.com/hastagAB), [Kaushlendra Pratap](https://github.com/Kaushl2208), and [Shaheem Azmal M MD.](https://github.com/shaheemazmalmmd) Their unwavering support and invaluable guidance have been instrumental in shaping this project. Whenever I sought advice or assistance, they generously shared their expertise and insights. Their commitment to mentoring has been a driving force behind my growth and the success of this venture.


