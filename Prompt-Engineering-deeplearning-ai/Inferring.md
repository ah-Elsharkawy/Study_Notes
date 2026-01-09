# Inferring

Inferencing is the process of using a trained machine learning model to make predictions on new, unseen data. This is a crucial step in deploying machine learning models in real-world applications, as it allows the model to provide insights and make decisions based on the patterns it has learned during training.
In the lecture they said its the process of using a trained model to analyze a text like extracting information, classifying it, or asking questions about it.

> [!note] Example of Inferencing
>
> The example in the video was a text which is a customer review of a product, and the instructor kept giving prompts to the model asking if the review is positive or negative, also he asked the model to extract specific information like te wrds that shows feelings in the review, also asked about the product name and the brand.
>
> - Then he merged all the previous prompts into one prompt and asked the model to do all the tasks in one go and format it as json object.
> - The second example was a news long article and he prompted the model to get te topics mentioned in the article seperated by commas for later plitting in python, after this he reversed the process and gave the model the topics and asked it to list these topics with value 0 if they are not mentioned in the article and 1 if they are mentioned, and he called this process 0 shot model, `because the model was able to do this task without any training on it`.
