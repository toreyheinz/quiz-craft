# Assessment Generation Instructions for Quiz Craft

## Overview
Generate a JSON assessment file that follows the Quiz Craft format. The assessment should be educational, well-structured, and include explanations for deeper learning.

## Required JSON Structure

```json
{
  "title": "Assessment Title",
  "description": "Brief description of the assessment (1-2 sentences)",
  "subjects": [
    {
      "name": "Subject Name",
      "questions": [
        {
          "id": 1,
          "question": "Clear, concise question text?",
          "options": ["Option A", "Option B", "Option C", "Option D"],
          "correctAnswer": 0,
          "correctAnswerText": "Optional: The actual answer text if different from the option",
          "explanation": "Optional but recommended: 1-2 sentence explanation of why this is correct"
        }
      ]
    }
  ]
}
```

## Field Specifications

### Required Fields:
- **title**: The assessment name (e.g., "Environmental Science Assessment")
- **description**: Brief overview of what the assessment covers
- **subjects**: Array of subject categories
- **name**: Subject category name (e.g., "Climate Change", "Ecosystems")
- **questions**: Array of questions for each subject
- **id**: Unique number for each question (sequential: 1, 2, 3...)
- **question**: The question text (end with "?" for questions)
- **options**: Exactly 4 answer choices as an array of strings
- **correctAnswer**: Index of the correct option (0-based: 0=first, 1=second, 2=third, 3=fourth)

### Optional Fields:
- **correctAnswerText**: Use when the answer needs clarification beyond the option text
- **explanation**: Educational explanation shown when students answer incorrectly

## Content Guidelines

### Question Quality:
1. **Clear and Concise**: Questions should be unambiguous and direct
2. **Single Concept**: Each question should test one specific concept
3. **Appropriate Level**: Match the difficulty to the target grade level
4. **Factual Accuracy**: Ensure all information is correct and up-to-date
5. **No Trick Questions**: Avoid misleading or confusing wording

### Answer Options:
1. **Plausible Distractors**: Wrong answers should be reasonable but clearly incorrect
2. **Similar Length**: Keep all options roughly the same length
3. **Grammatically Consistent**: All options should fit grammatically with the question
4. **No "All/None of the Above"**: Avoid these option types
5. **Randomized Correct Position**: Don't always make the first or last option correct

### Explanations (Highly Recommended):
1. **Educational Value**: Explain why the answer is correct
2. **Address Misconceptions**: Clarify common misunderstandings
3. **Provide Context**: Add relevant facts or connections
4. **Keep Concise**: 1-2 sentences is usually sufficient

## Distribution Guidelines

### Question Distribution:
- Divide questions evenly across subjects when possible
- For 20 questions with 4 subjects: 5 questions each
- For 24 questions with 4 subjects: 6 questions each
- Adjust as needed for odd numbers

### Difficulty Balance:
- 30% Easy (fundamental concepts)
- 50% Medium (application of knowledge)
- 20% Challenging (analysis/synthesis)

## Example Assessment Structure

```json
{
  "title": "Ancient Civilizations Assessment",
  "description": "Test your knowledge of major ancient civilizations and their contributions to human history.",
  "subjects": [
    {
      "name": "Ancient Egypt",
      "questions": [
        {
          "id": 1,
          "question": "Which river was essential to ancient Egyptian civilization?",
          "options": ["Tigris River", "Nile River", "Euphrates River", "Ganges River"],
          "correctAnswer": 1,
          "explanation": "The Nile River provided water, fertile soil through annual floods, and transportation, making Egyptian civilization possible in the desert."
        },
        {
          "id": 2,
          "question": "What writing system did ancient Egyptians use?",
          "options": ["Cuneiform", "Hieroglyphics", "Sanskrit", "Latin alphabet"],
          "correctAnswer": 1,
          "explanation": "Hieroglyphics was the formal writing system of ancient Egypt, using pictorial symbols to represent words and sounds."
        }
      ]
    },
    {
      "name": "Ancient Greece",
      "questions": [
        {
          "id": 3,
          "question": "Which city-state is known as the birthplace of democracy?",
          "options": ["Sparta", "Corinth", "Athens", "Thebes"],
          "correctAnswer": 2,
          "explanation": "Athens developed the world's first democracy around 508 BCE under the leadership of Cleisthenes."
        }
      ]
    }
  ]
}
```

## Best Practices

1. **Validate JSON**: Ensure proper JSON formatting with no syntax errors
2. **Sequential IDs**: Number questions from 1 to total count
3. **Consistent Formatting**: Maintain consistent capitalization and punctuation
4. **Review Content**: Double-check facts, spelling, and grammar
5. **Test Logic**: Verify correctAnswer indices match the intended correct options

## Common Mistakes to Avoid

1. Using 1-based indexing for correctAnswer (use 0-based instead)
2. Missing commas between array elements or object properties
3. Including fewer or more than 4 options
4. Duplicate question IDs
5. Mismatched correctAnswer index
6. Questions without question marks
7. Explanations that give away answers to other questions

## Output Requirements

Generate ONLY the JSON assessment without any additional text, markdown formatting, or explanations outside the JSON structure. The output should be valid JSON that can be directly uploaded to Quiz Craft.