<a name="top"></a>
# Please View in Colab To See Figures!!!

# Quick Research Summary
## Preface
*  This is a hackathon-esq notebook demonstraing how I go about analyzing model behaviors and interpret figures. I often set 10 hour limits when doing such searches as there may be other behaviors more worth while exploring!
*   Hopefully this notebook demonstrated my ability to analyze behaviors in the wild.
*   Note: Under every chart linked in this summary there is a more thorough interpretation underneath the chart than what I provide in the summary.


## Initial exploration
  *   In playing around with GPT2-Small I noticed that it would repeat some grammatical patterns. The one I decided to focus on was using contractions that previously appeared in sentences.
      *   "We're helping our friend while they" predicts "'re" instead "are"
      *   "We are helping our friend while they" predicts "are" instead "'re"
  *   I also noticed that the model will (mostly) continue to use contractions even if the final contraction is distinct from the previously used one.
    *   "We're helping our friend while he" predicts 's instead "is" despite the previous contraction in the sentence being 're

## Research Goal
*   For one type of contraction earlier in the sentence paired with one terminating pronoun, how does the model know to predict the correct contraction?
*   For several different contraction pronoun pairs, can I identify the model's mechanisms used to predict the final contraction?
*   If I can do the above, can I compare the various mechanisms used to predict contractions? Are they the same? Are they different? If so how are they different? Do they use various parts of the same circuit?
*   These questions are super exciting to me yet are probably too grand in scope for this application. I set out to make as much progress in answering these questions as I can while learning the tools of mechanistic interpretability.

## Hypothesis
*   From purely thinking about what the model could be implementing, I think it is possible that the mechanism to do this task involves Bigram/trigram munging. Bigrams are when the model predicts the next token based on what is most likely to appear after exclusively the final token. Skip trigrams fare when the model sees some pattern "A...B" and then predicts C. I believe it is possible that the model looks at the pronoun at the end of the sentence, boosts verbs and verb contractions, and from the trigram with the earlier contraction token, boosts the correct contraction of verbs for that pronoun.
*   I anticipate that the mechanisms used to do this are slightly different for various contraction pronoun pairs. I am very uncertain about this but my reasoning is that the model (in some contraction pairings) will be able to copy the previous contraction instead of relying only on the bigram with the pronoun at the end. I wonder if the model will favor copying information from the contraction in some cases or will just stick to the same general mechanism.

## Experiment 1: Looking at the ('re, they) pairing
*   By decomposing the residual stream layer by layer [we can see](#E1:LA)  that the most the most important parts of the model for this task are layer 7 attention and layer 8 and 10 MLP.
*   Looking at the [attention heads](#E1:AH) we see that L7 H11 seems critically important to completing the task. Inspecting it further we see that [this head](#E1:HP) attends the contraction token to the final token
*   I then conducted a ROME like analysis with corrupted prompts to further look at this behavior
*   By [patching attention layer activations](#E1:ALP) we see that we can recover a good amount of performance on corrupted prompts by patching L7 and L8 which aligns with our previous analysis but interestingly we can't fully recover performance which means these
*   By [patching MLP layers](#E1:MLPP) we see that we can recover a surprising amount from MLP layers 1 3 8 and 10.
*   As explained [below](#neurExp), I wanted to inspect all the neurons in these layers on neuroscope but realized this was simply too many neurons to look at. Instead I used [clementneo's](#credit) technique for activation patching for individual neurons. I was hoping this way I could narrow down my search for neurons (recognizing that I was overlooking important neurons) that would be important on their own as a starting point. Due to time I only did this for [layer 8](#E1:NP8) and [layer 10](#E1:NP10). I found [Layer: 8. Neuron Index: 2744](#E1:NI2744), [Layer: 10. Neuron Index: 1063](#E1:NI1063), and [Layer: 10 . Neuron Index: 2193](#E1:NI2193) to be important. Their corresponding neuroscope pages also made sense for why this would be the case!
*   Lastly I decomposed the heads and found that the [value patching](#E1:VPP) most significantly recovered performance in L7 H11 and not the [attention pattern patching](#E1:APP) which contradicted what I thought was going on when I visualized this head. Would want to look more closely into why this is the case! We [can see](#E1:HP) that there are other heads (L10 H1) in the model that attend the contraction token to the final one yet maybe they don't copy the same information as head 7 which would explain why the value patching is more important than the pattern patching. Will look more in depth as to the plausibility of this after the application is due but can't say this is definitely the case without further analysis.

## Experiment 2: Looking at other pairings

*   I won't go as in depth into the analysis itself because it pretty closely aligns with the analysis I did above just for multiple experimental trials where the prompts within a trial all have the same contraction pronoun pairing.
*   The various prompts and experimental groups for each pairing can be found [here](#E2:DPA)
*   [Decomposing the residual stream](#E2:LA) layer by layer suggests that for all pronoun pairings the same 3 layers are the most important yet which one of those layers is the most important varies slightly
*   We can see by looking at the [heads](#E2:HA) that L7 H11 is by far the most important again and the other top heads are largely the same yet for the 's we, 's they pairings L11 H8 was the third most important head which is not the case for the other heads.
*   I then conducted an analysis for each experimental group with corrupted prompts and found that the [same layers could improve performance in each experimental group](#E2:RSP)
*   Finally I [Attention Layers](#E2:ALP) and [patched MLP Layers](#E2:MLPP) from the clean runs of the model
*   For Attention Layers layers I noticed that largely the same layers are important for each task but there is variation! For 're he and 're she pairings, L8 is slightly more significant that L7 yet for some pairings like 's she and 's he L8 is less significant than L7. Granted these differences are < 0.2 which may not be significant but it is there!
*   For the MLP Layers there is slightly more variation in the later layers between the tasks. The differences between the experimental groups (largerly the use of L11 MLP) can be largely bundled into two groups categorized by what the predicted contraction should be. Within each group that chart looks largely similar. Again these differences are within a small range < 0.2 so may not be significant but are definitely interesting!


# Wrap-up
*   As explained above, I believe that some of the observations suggest that maybe there is some trigram bigram munging happening to perform this task but I would be far from saying that I can reject any sort of null hypothesis.
*   This emphasizes to me just how complicated and involved some of the processing this model is doing actually is. A good example is telling myself to keep looking into MLPs eventhough it seemed like L7 H11 provided a simple all encompassing solution. I am now certain the truthful circuit is far more involved than a simple attention pattern.

















