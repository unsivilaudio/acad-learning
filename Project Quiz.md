# Project Quiz

## Tasks

1. **Build Quiz component**
   - should handle all the state of your quiz application
   - should iterate through the quiz questions
   - display a results screen when all quiz questions have been exhausted
1. **Build a Question component**
   - displays the main question text
   - displays an list of answers (Answers component)
     <br />
     _[optional][advanced]_
     - give each question a limited amount of time for the user to answer before skipping the question (save 'null' result for user answer)
     - abstract timer logic to its own component called QuestionTimer
1. **Build an Answers component**
   - displays a list of clickable answers
   - implement when an answer is selected will move to the next question in the list
1. **Build a Summary component**
   - should be shown (in the Quiz component) when all answers in the quiz have been exhausted
   - iterate through the answers, and display the question text, and the user answer
   - user answer should reflect if correct (green), incorrect (red), or "skipped", not-answered (white)
     - [optional] - should show 3 statistics (in percentage), "skipped" (no answer), "correct", "incorrect"
1. [optional] - **Build a Header component**
   - should be displayed at the top of your page

#### Hints

- you will need to handle lifting state up through (sometimes multiple) components to keep your application state in the Quiz component
- you can use the questions length to calculate your statistics (the results below are multiplied by 100 to give us a percentage)
  - num-of-correct / questions.length x 100
  - num-of-incorrect / questions.length x 100
  - num-of-skipped / questions.length x 100
- minimize your state values in Quiz, and derive as much of your application values as possible
- don't forget to properly memoize any functions/values you might need to reference in effect dependency arrays
  - useCallback for functions
  - useMemo for values
