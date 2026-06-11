---
cssclasses:
  - dashboard
---

```dataviewjs

const vaultName = "YOUR VAULT NAME";

let today = new Date();
let estOffset = -new Date().getTimezoneOffset();
let todayString = new Date(today.getTime() + estOffset * 60 * 1000).toISOString().split("T")[0];

let startOfWeek = new Date(today);
startOfWeek.setDate(today.getDate() - today.getDay());

let calendar = Array.from({ length: 56 }, () => []);

let allPages = dv.pages().filter(p => p.file);

let tasks = [];
allPages.forEach(p => {
    if (p.file && p.file.tasks) {
        tasks.push(
            ...p.file.tasks
                .filter(t => !t.completed)
                .map(task => ({
                    ...task,
                    fileName: p.file.name,
                    filePath: p.file.path,
                }))
        );
    }
});

let holidayDates = [];

dv.pages('"events"').forEach(p => {
    if (p.file && p.file.tasks) {
        p.file.tasks
            .filter(t => t.text.includes("#tasks/holiday"))
            .forEach(t => {
                let dateRange = t.text.match(/\b\d{4}-\d{2}-\d{2}\b/g);
                if (dateRange && dateRange.length === 2) {
                    let startDate = new Date(dateRange[0]);
                    let endDate = new Date(dateRange[1]);
                    while (startDate <= endDate) {
                        holidayDates.push(startDate.toISOString().split("T")[0]);
                        startDate.setDate(startDate.getDate() + 1);
                    }
                } else {
                    let singleDate = dateRange ? new Date(dateRange[0]) : null;
                    if (singleDate) {
                        holidayDates.push(singleDate.toISOString().split("T")[0]);
                    }
                }
            });
    }
});

if (tasks.length === 0) {
    dv.paragraph("No tasks found in the vault.");
} else {
    let taskMap = {};

    tasks.forEach(t => {
        let taskDate = extractDateFromText(t.text);
        if (taskDate) {
            let dateObj = new Date(taskDate);

            for (let day = 0; day < 56; day++) {
                let currentDate = new Date(startOfWeek);
                currentDate.setDate(startOfWeek.getDate() + day);

                let adjustedDate = new Date(currentDate.getTime() + estOffset * 60 * 1000);

                if (dateObj.toISOString().split("T")[0] === adjustedDate.toISOString().split("T")[0]) {
                    if (!taskMap[taskDate]) {
                        taskMap[taskDate] = [];
                    }

                    
                    let taskText = t.text
                        .replace(/\b\d{4}-\d{2}-\d{2}\b/g, "") 
                        .replace(/#\S+/g, "") 
                        .replace(/[\u{1F300}-\u{1F6FF}]/gu, "") 
                        .replace(/::/g, "") 
                        .replace(/every.*/gi, "") 
                        .trim();

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


let taskWithLink = `<a href="obsidian://open?vault=${encodeURIComponent(vaultName)}&file=${encodeURIComponent(t.filePath)}#${encodeURIComponent(t.text)}" style="color: white; text-decoration: none;">${taskTextStyled}</a>`;


                    taskMap[taskDate].push(taskWithLink);
                }
            }
        }
    });

    for (let day = 0; day < 56; day++) {
        let currentDate = new Date(startOfWeek);
        currentDate.setDate(startOfWeek.getDate() + day);

        let adjustedDate = new Date(currentDate.getTime() + estOffset * 60 * 1000);
        let dateDisplay = adjustedDate.toISOString().split("T")[0];
        calendar[day] = taskMap[dateDisplay] || [];
    }

    let calendarHTML = "";

    for (let row = 0; row < 8; row++) {
        calendarHTML += '<div style="display: flex; flex-direction: row; justify-content: space-between; margin-bottom: 10px;">';

        for (let day = 0; day < 7; day++) {
            let currentIndex = row * 7 + day;
            let currentDate = new Date(startOfWeek);
            currentDate.setDate(startOfWeek.getDate() + currentIndex);

            let adjustedDate = new Date(currentDate.getTime() + estOffset * 60 * 1000);
            let dayNumber = adjustedDate.getDate();
            let monthName = adjustedDate.toLocaleString("default", { month: "short" });
            let dateDisplay = `${dayNumber} ${monthName}`;

            let cellContent = calendar[currentIndex]
                .map(task => `<div style="font-size: 12px; overflow: hidden; white-space: nowrap; text-overflow: ellipsis;">${task}</div>`)
                .join("");

            let isToday = adjustedDate.toISOString().split("T")[0] === todayString;
            let isHoliday = holidayDates.includes(adjustedDate.toISOString().split("T")[0]);
            let bgColor = isHoliday ? "#000080" : isToday ? "#202020" : "#303030";

            calendarHTML += `<div style="font-size: 12px; background-color: ${bgColor}; color: #A020F0; padding: 10px; width: 14%; border: 1px solid #444; margin: 2px; border-radius: 5px; text-align: center;">` +
                `<div style="font-weight: bold;">${dateDisplay}</div>${cellContent}</div>`;
        }
        calendarHTML += "</div>";
    }

    dv.paragraph(calendarHTML);
}

function extractDateFromText(text) {
    const datePattern = /\b\d{4}-\d{2}-\d{2}\b/;
    const match = text.match(datePattern);
    return match ? match[0] : null;
}
```


























