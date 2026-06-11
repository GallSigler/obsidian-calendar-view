# Set-up 

1. Make sure you have the Dataview plug-in turned on, as well as the "Enable JavaScript queries"
2. Copy `calendar-view.md` into your vault
3. Define your vault name here:

```
const vaultName = "YOUR VAULT NAME";
```

4. Change `tag1`, `tag2`, and `tag3` with the tags you use in your vaults:

```
let taskTextStyled;
if (t.text.includes("#tasks/deadline")) {
    taskTextStyled = `<strong style="color: red;">${taskText}</strong>`;
} else if (t.text.includes("#tasks/tag1")) {
    taskTextStyled = `<span style="background-color: #2C3E43; color: black; border-radius: 3px; padding: 1px 3px;">${taskText}</span>`;
} else if (t.text.includes("#tasks/tag2")) {
    taskTextStyled = `<span style="background-color: #7A8B7B; color: black; border-radius: 3px; padding: 1px 3px;">${taskText}</span>`;
} else if (t.text.includes("#tasks/tag3")) {
    taskTextStyled = `<span style="background-color: #B89075; color: black; border-radius: 3px; padding: 1px 3px;">${taskText}</span>`;
 } else {
    taskTextStyled = taskText;
}
```

Feel free to add in the same manner more colors for other tags

# Example 



