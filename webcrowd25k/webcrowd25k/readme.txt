WebCrowd25k dataset
This dataset is released for research use only.

URLs (mirrored):
* http://ir.ischool.utexas.edu/webcrowd25k/
* http://qufaculty.qu.edu.qa/telsayed/datasets/webcrowd25k/

Publications (to cite for use of this dataset)
------------
* Tanya Goyal, Tyler McDonnell, Mucahid Kutlu, Tamer Elsayed and Matthew Lease. Your Behavior Signals Your Reliability: Modeling Crowd Behavioral Traces to Ensure Quality Relevance Annotations. In Proceedings of the 6th AAAI Conference on Human Computation and Crowdsourcing (HCOMP), pages 41-49, 2018.
* Mucahid Kutlu, Tyler McDonnell, Yassmine Barkallah, Tamer Elsayed and Matthew Lease.  Crowd vs. Expert: What Can Relevance Judgment Rationales Teach Us About Assessor Disagreement? In Proceedings of the 41st international ACM SIGIR conference on Research and development in Information Retrieval (SIGIR), pages 805-814, 2018.

Key Prior Publications
----------------------
* Tyler McDonnell, Matthew Lease, Mucahid Kutlu, and Tamer Elsayed. Why Is That Relevant? Collecting Annotator Rationales for Relevance Judgments. In Proceedings of the 4th AAAI Conference on Human Computation and Crowdsourcing (HCOMP), pages 139-148, 2016. Best Paper Award.
* Brandon Dang, Miles Hutson, and Matthew Lease. MmmTurkey: A Crowdsourcing Framework for Deploying Tasks and Recording Worker Behavior on Amazon Mechanical Turk. In 4th AAAI Conference on Human Computation and Crowdsourcing (HCOMP): Works-in-Progress Track, 2016. 3 pages. arXiv:1609.00945.

====================================================================

The dataset includes three related parts:

* Crowd Relevance Judgments. 25,099 information retrieval relevance judgments collected on Amazon's Mechanical Turk platform. For each of the 50 search topics from the 2014 NIST TREC WebTrack, we selected 100 ClueWeb12 documents to be re-judged (without reference to the original TREC assessor judgment) by 5 MTurk workers each (50 topics x 100 documents x 5 workers = 25K crowd judgments). Individual worker IDs from the platform are hashed to new identifiers. We collect relevance judgments on a 4-point graded scale. (See SIGIR'18 & HCOMP'18 papers).

* Behavioral Data. For a subset of the judgments, we also collected behavioral data charactering worker behavior in performing the relevance judging. Behavioral data was recorded using MmmTurkey, which captures a variety of worker interaction behaviors while completing MTurk Human Intelligence Tasks. (See HCOMP'18 paper)

* Disagreement Analysis. We inspected 1000 crowd judgments for 200 documents (5 judgments per document, where the aggregated crowd judgment differs from the original TREC assessor judgment), and we classified each disagreement according to our disagreement taxonomy. (See SIGIR'18 paper.)

The data release contains 4 files:

1) gold_judjements.txt: This file contains the gold NIST judgements for all (document,topic) pairs.
2) crowd_judgements.csv: This file contains the crowd relevance responses for 25,000 (document,topic) pairs
3) behaviorDataRelease.json: This file contains the behavior data collected for the crowd workers
4) disagreement_analysis.xlsx: This file contains the NIST, crowd, and authors' annotations and disagreement analysis of 200 documents.

The contents of the files are explained in more details below.

1) gold_judjements.txt
	Each row corresponds to the gold judgement for a specific (document, topic) pair. This is a space delimited file with 4 columns. The columns are (in order):
	Topic ID - Web Track 2014 topic ID for the topic (document, topic) pair.
	Unused column - This column is same across all rows, and should be ignored.
	Document ID - ClueWeb12 identifier for the document in the (document, topic) pair.
	Gold Judgement - The gold relevance judgement of the document with respect to the topic. This columns takes values {-2, 0, 1, 2, 3, 4}
					{-2: This page does not appear to be useful for any reasonable purpose; it may be spam or junk.
					  0: The content of this page does not provide useful information on the topic.
					  1: The content of this page provides some information on the topic.
					  2: The content of this page provides substantial information on the topic.
					  3: This page or site is dedicated to the topic; authoritative and comprehensive, it is worthy of being a top result in a web search engine.
					  4: This page represents a home page of an entity directly named by the query; the user may be searching for this specific page or site.}

2) crowd_judgements.csv
	This file records the crowd relevance responses for 25k judgments. Each row in the csv corresponds to the relevance response of one worker, for a specific (document, topic) pair. This is a csv file with the following columns:
	ID: An identifier for the row. Can be ignored.
	tid: Web Track 2014 topic ID for the topic (document, topic) pair.
	did: ClueWeb12 identifier for document of the (document, topic) pair.
	mapping: A randomly generated string for each HIT. This mapping ID is used to map HITs to the corresponding behavioral data released in file "behaviorDataRelease.json".
	wid: Anonymized worker ID for the mechanical turk worker
	start: Date time at which the task was started by the worker.
	duration: : Total time in seconds spent by worker on task.
	label: Relevance label provided by the worker. { 0 = Definitely Not Relevant, 1 = Probably Not Relevant, 2 = Probably Relevant, 3 = Definitely Relevant} (Some label values 	   may be -1, this signifies that the worker did not provide any response, and such responses should be ignored).
	rationale: Text rationale provided by worker for their relevance judgment.
	design: This column is same acros all rows, and should be ignored.
	feedback: Optional freeform task feedback provided by worker.

	The crowd judgments can be mapped to the gold judgements from "gold_judjements.txt" by matching the topic id and document id pairs.

3) behaviorDataRelease.json
	This json file contains the behavioral data of the crowd workers. The json file contrains (key, value) pairs. The key of the json object contains mapping IDs. The value describes the behavior data. One must match this mapping ID to the "mapping" column of "crowd_judgements.csv" to establish a mapping between HITs and it's corresponding worker behavior data.
	
4) disagreement_analysis.xlsx
	This excel file contains detailed annotations and disagreement analysis of 200 documents. It has two sheets: README and Data. Please refer to the README sheet for more information about how to understand the data and the analysis.