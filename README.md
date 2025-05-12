# Survey Form Builder (ReactJS)
## Date:12-05-2025

## AIM
To create a Survey Form Builder using ReactJS..

## ALGORITHM
### STEP 1 Initialize States
mode ← 'build' (default mode)

questions ← [] (holds all survey questions)

currentQuestion ← { text: '', type: 'text', options: '' } (holds input while building)

editingIndex ← null (tracks which question is being edited)

responses ← {} (user responses to survey)

submitted ← false (indicates whether survey was submitted)

### STEP 2 Switch Modes
Toggle mode between 'build' and 'fill' when the toggle button is clicked.

Reset submitted to false when entering Fill Mode.

### STEP 3 Build Mode Logic
#### a. Add/Update Question
If currentQuestion.text is not empty:

If type is "radio" or "checkbox", split currentQuestion.options by commas → optionsArray.

Construct a questionObject:

If editingIndex is not null, replace question at that index.

Else, push questionObject into questions.

Reset currentQuestion and editingIndex.

#### b. Edit Question
Set currentQuestion to the selected question’s data.

Set editingIndex to the question’s index.

#### c. Delete Question
Remove the selected question from questions.

### STEP 4 Fill Mode Logic
#### a. Render Form
For each question in questions:

If type is "text", render a text input.

If type is "radio", render radio buttons for each option.

If type is "checkbox", render checkboxes.

#### b. Capture Responses
On input change:

For "text" and "radio", update responses[index] = value.

For "checkbox":

If option exists in responses[index], remove it.

Else, add it to the array.

### STEP 5 Submit Form
When user clicks "Submit":

Set submitted = true.

Display a summary using the questions and responses.

### STEP 6 Display Summary
Iterate over questions.

Display the response from responses[i] for each question:

If it's an array, join it with commas.

If empty, show "No response".


## PROGRAM
APP.JS
```
import React, { useState } from 'react';
import './App.css';

function App() {
  const [mode, setMode] = useState('build');
  const [questions, setQuestions] = useState([]);
  const [currentQuestion, setCurrentQuestion] = useState({ text: '', type: 'text', options: '' });
  const [editingIndex, setEditingIndex] = useState(null);
  const [responses, setResponses] = useState({});
  const [submitted, setSubmitted] = useState(false);

  const handleAddOrUpdateQuestion = () => {
    if (!currentQuestion.text.trim()) return;

    const questionObj = {
      ...currentQuestion,
      options: ['radio', 'checkbox'].includes(currentQuestion.type)
        ? currentQuestion.options.split(',').map(opt => opt.trim())
        : []
    };

    const updatedQuestions = [...questions];
    if (editingIndex !== null) {
      updatedQuestions[editingIndex] = questionObj;
    } else {
      updatedQuestions.push(questionObj);
    }

    setQuestions(updatedQuestions);
    setCurrentQuestion({ text: '', type: 'text', options: '' });
    setEditingIndex(null);
  };

  const handleEditQuestion = index => {
    const q = questions[index];
    setCurrentQuestion({
      text: q.text,
      type: q.type,
      options: q.options?.join(',') || ''
    });
    setEditingIndex(index);
  };

  const handleDeleteQuestion = index => {
    const updated = [...questions];
    updated.splice(index, 1);
    setQuestions(updated);
  };

  const handleResponseChange = (index, value) => {
    setResponses(prev => ({ ...prev, [index]: value }));
  };

  const handleCheckboxChange = (index, option) => {
    const current = responses[index] || [];
    const updated = current.includes(option)
      ? current.filter(o => o !== option)
      : [...current, option];
    setResponses(prev => ({ ...prev, [index]: updated }));
  };

  const handleSubmit = e => {
    e.preventDefault();
    setSubmitted(true);
  };

  const handleModeSwitch = (newMode) => {
    setMode(newMode);
    setSubmitted(false);
  };

  return (
    <div className="container">
      <h1>Dynamic Survey Form Builder</h1>

      <div className="mode-toggle">
        <button onClick={() => handleModeSwitch('build')} className={mode === 'build' ? 'active' : ''}>Build Mode</button>
        <button onClick={() => handleModeSwitch('fill')} className={mode === 'fill' ? 'active' : ''}>Fill Mode</button>
      </div>

      {mode === 'build' && (
        <div className="builder">
          <div className="question-form">
            <input
              type="text"
              placeholder="Question text"
              value={currentQuestion.text}
              onChange={e => setCurrentQuestion({ ...currentQuestion, text: e.target.value })}
            />
            <select
              value={currentQuestion.type}
              onChange={e => setCurrentQuestion({ ...currentQuestion, type: e.target.value })}
            >
              <option value="text">Text</option>
              <option value="radio">Radio</option>
              <option value="checkbox">Checkbox</option>
            </select>
            {(currentQuestion.type === 'radio' || currentQuestion.type === 'checkbox') && (
              <input
                type="text"
                placeholder="Options (comma-separated)"
                value={currentQuestion.options}
                onChange={e => setCurrentQuestion({ ...currentQuestion, options: e.target.value })}
              />
            )}
            <button className="add-btn" onClick={handleAddOrUpdateQuestion}>
              {editingIndex !== null ? 'Update Question' : 'Add Question'}
            </button>
          </div>

          <div className="question-list">
            <h3>Questions</h3>
            {questions.map((q, index) => (
              <div key={index} className="question-item">
                <div>
                  <strong>{q.text}</strong> ({q.type})
                  {q.options?.length > 0 && (
                    <ul>
                      {q.options.map((opt, i) => (
                        <li key={i}>{opt}</li>
                      ))}
                    </ul>
                  )}
                </div>
                <div>
                  <button onClick={() => handleEditQuestion(index)}>Edit</button>
                  <button onClick={() => handleDeleteQuestion(index)}>Delete</button>
                </div>
              </div>
            ))}
          </div>
        </div>
      )}

      {mode === 'fill' && questions.length > 0 && (
        <form onSubmit={handleSubmit} className="fill-form">
          {questions.map((q, index) => (
            <div key={index} className="form-question">
              <label>{q.text}</label>
              {q.type === 'text' && (
                <input
                  type="text"
                  value={responses[index] || ''}
                  onChange={e => handleResponseChange(index, e.target.value)}
                />
              )}
              {q.type === 'radio' &&
                q.options.map((opt, i) => (
                  <div key={i}>
                    <input
                      type="radio"
                      name={`q-${index}`}
                      value={opt}
                      checked={responses[index] === opt}
                      onChange={() => handleResponseChange(index, opt)}
                    />
                    <label>{opt}</label>
                  </div>
                ))}
              {q.type === 'checkbox' &&
                q.options.map((opt, i) => (
                  <div key={i}>
                    <input
                      type="checkbox"
                      value={opt}
                      checked={responses[index]?.includes(opt) || false}
                      onChange={() => handleCheckboxChange(index, opt)}
                    />
                    <label>{opt}</label>
                  </div>
                ))}
            </div>
          ))}
          <button type="submit">Submit</button>
        </form>
      )}

      {submitted && (
        <div className="summary">
          <h2>Survey Summary</h2>
          {questions.map((q, index) => (
            <div key={index}>
              <strong>{q.text}:</strong>{' '}
              {Array.isArray(responses[index])
                ? responses[index].join(', ')
                : responses[index] || 'No response'}
            </div>
          ))}
        </div>
      )}
    </div>
  );
}

export default App;

```
APP.CSS
```
.container {
  max-width: 800px;
  margin: auto;
  padding: 20px;
  font-family: Arial, sans-serif;
}

h1 {
  text-align: center;
}

.mode-toggle {
  text-align: center;
  margin-bottom: 20px;
}

.mode-toggle button {
  padding: 10px 20px;
  margin: 5px;
  font-size: 16px;
}

.mode-toggle .active {
  background-color: #007bff;
  color: white;
}

.builder,
.fill-form,
.summary {
  background: #f7f7f7;
  padding: 20px;
  border-radius: 8px;
}

.question-form input,
.question-form select {
  display: block;
  margin-bottom: 10px;
  padding: 8px;
  width: 100%;
  font-size: 16px;
}

.add-btn {
  background-color: green;
  color: white;
  padding: 10px;
  font-size: 16px;
  border: none;
}

.question-item {
  border: 1px solid #ccc;
  padding: 10px;
  margin: 10px 0;
}

.form-question {
  margin-bottom: 20px;
}

```


## OUTPUT
![Screenshot (253)](https://github.com/user-attachments/assets/a928b7e1-c61c-4fa5-b5aa-6ff190dfbe16)

![Screenshot (254)](https://github.com/user-attachments/assets/83213115-b9ce-4221-ad50-f35c516e6387)


## RESULT
The program for creating Survey Form Builder using ReactJS is executed successfully.
