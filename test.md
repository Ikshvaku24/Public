Here are 10 MCQ questions for your post-assessment, covering Day 1 (Python Fundamentals) and Day 2 (Pandas):

---

### 🐍 Day 1 — Python Fundamentals (Q1–Q5)

---

**Q1. What will the following code print?**
```python
fruits = ["apple", "banana", "cherry"]
print(fruits[-2])
```
- A) apple
- B) banana ✅
- C) cherry
- D) IndexError

---

**Q2. Which of the following correctly creates a dictionary comprehension that maps numbers 1–5 to their squares?**

- A) `{i: i**2 for i in range(1, 6)}` ✅
- B) `[i: i**2 for i in range(1, 6)]`
- C) `{i**2 for i in range(1, 6)}`
- D) `(i: i**2 for i in range(1, 6))`

---

**Q3. What is the output of this code?**
```python
def add(a, b=10):
    return a + b

print(add(5))
print(add(5, 3))
```
- A) 15 and 8 ✅
- B) 5 and 8
- C) 15 and 15
- D) Error — missing argument

---

**Q4. A student runs the following. What happens and why?**
```python
coordinates = (10.5, 20.3)
coordinates[0] = 15
```
- A) coordinates becomes (15, 20.3)
- B) Only the first element updates
- C) TypeError — tuples are immutable ✅
- D) ValueError — cannot assign float to tuple

---

**Q5. What will the output of this exception handling block be?**
```python
try:
    result = int("abc")
except ValueError:
    print("Caught ValueError")
except TypeError:
    print("Caught TypeError")
finally:
    print("Done")
```
- A) Caught TypeError → Done
- B) Caught ValueError ✅ (only)
- C) Caught ValueError → Done ✅
- D) The program crashes with no output

> *(Correct answer: C — both "Caught ValueError" and "Done" print)*

---

### 🐼 Day 2 — Pandas & Data Processing (Q6–Q10)

---

**Q6. You have this DataFrame `df`. What does the following return?**
```python
df[df['Salary'] > 70000]['Department'].value_counts()
```
- A) All departments and their average salary
- B) Count of employees per department where salary > 70000 ✅
- C) A filtered DataFrame with only the Department column
- D) Error — cannot chain filters like this

---

**Q7. What is the difference between `dropna()` and `fillna()` in Pandas?**

- A) `dropna()` removes duplicate rows; `fillna()` removes null rows
- B) `dropna()` removes rows/columns with null values; `fillna()` replaces nulls with a specified value ✅
- C) Both do the same thing — they remove null values
- D) `fillna()` only works on numeric columns

---

**Q8. What will `df.groupby('Region')['Sales'].agg(['sum', 'mean'])` produce?**

- A) One row per Region showing total and average Sales ✅
- B) A single scalar — the overall sum and mean of Sales
- C) Error — `agg()` only accepts a single function
- D) A pivot table with Regions as columns

---

**Q9. A colleague runs `pd.merge(df1, df2, on='ID', how='left')`. Which statement is correct?**

- A) Only rows where ID exists in BOTH tables are kept
- B) All rows from df2 are kept; matching rows from df1 are added
- C) All rows from df1 are kept; matching data from df2 is added; unmatched df2 rows are dropped ✅
- D) All rows from both tables are kept regardless of match

---

**Q10. What is the correct way to extract the month from a datetime column `order_date` in Pandas?**

- A) `df['order_date'].month`
- B) `df['order_date'].dt.month` ✅
- C) `df['order_date'].datetime.month`
- D) `pd.month(df['order_date'])`

---

### 📊 Answer Key (for your reference)

| Q | Answer | Topic |
|---|--------|-------|
| 1 | B | Lists & negative indexing |
| 2 | A | Dictionary comprehensions |
| 3 | A | Functions with default parameters |
| 4 | C | Tuples — immutability |
| 5 | C | Exception handling + finally |
| 6 | B | Filtering + value_counts |
| 7 | B | Handling null values |
| 8 | A | GroupBy + agg |
| 9 | C | Merge — left join |
| 10 | B | DateTime `.dt` accessor |