# TextrankSummary

This architecture uses a LSTM (a tree-shaped variation on a recurrent neural network, described in https://arxiv.org/pdf/1503.00075.pdf) to compute vectorized representations of phrases and sentences. Their distances are then used as distance between sentence nodes in the main TextRank.

As opposed to the aforementioned article, where the distance metric incorporates the angle and the absolute distance of sentence vectors as parameters, this one will use a (learned) neural network from a vector space of size 3n+1, incorporating both vectors as well as their dot product as parameters, as well as the topic vector (more on this later). This allows for a more flexible distance metric, able to place emphasis on relevant differences and similarities between the sentences.

Here, n is the size of the cleaned output: that is, the vectorized output of the recurrent neural network, from which the **most significant** axis (in the SVD of the matrix of sentence vectors) has been removed (suprisingly, this has been proved to have a positive effect on the data).

Another innovation is the topic vector. This vector represents the main topic of the sentence, and is produced by sending through a recurrent neural network the raw sentence vectors. (Note: Learning process for the topic vector is yet unclear, and, if a good database is lacking, it may revert to a simple average of the sentence vectors).

The main algorithm used is a modified TextRank, with weights both on the undirected edges (corresponding to the distance between sentences, averaged in both directions to prevent inconsistencies) and on the vertices (corresponding to the distance between a sentence and the topic).

With this, the most relevant sentences are selected and incorporated into the summary.

Once all three (two?) neural networks are properly trained, the summarization process is thus:
  - Parse the text into sentences using a regex
  - Parse the sentences into a labelled tree structure using a Huggingface model
  - Find x-bar schemas, flatten them and label them
  - Run each sentence through the (already trained) LSTM to compute sentence vectors
  - Compute the topic vector
  - Perform SVD and clean the sentence and topic vectors
  - Compute the distances between sentences
  - Build the TextRank graph
  - Run TextRank to compute sentence relevance
  - Select the most relevant sentences up to the summary word limit
