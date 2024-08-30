---
title: Export Survey Results to PDF or Excel | Open-Source JS Form Builder
description: Convert your survey data to manageable table format for easy filtering and analysis. Save survey results as PDF or Excel files to visualize or share with others. View free demo for JavaScript with a step-by-step setup guide.
---

# Table View for Survey Results in a JavaScript Application

This step-by-step tutorial will help you set up a Table View for survey results using SurveyJS Dashboard an application built with HTML, CSS, and JavaScript (without frontend frameworks). As a result, you will create the view displayed below:

<iframe src="/proxy/github/code-examples/dashboard-table-view/html-css-js/index.html"
    style="width:100%; border:0; border-radius: 4px; overflow:hidden;"
></iframe>

[View Full Code on GitHub](https://github.com/surveyjs/code-examples/tree/main/dashboard-table-view/html-css-js (linkStyle))

## Link Resources

SurveyJS Dashboard depends on other JavaScript libraries. Reference them on your page in the following order:

1. Survey Core       
A platform-independent part of [SurveyJS Form Library](https://surveyjs.io/form-library/documentation/overview) that works with the survey model. SurveyJS Dashboard requires only this part, but if you also display the survey on the page, reference [the rest of the SurveyJS Form Library resources](/Documentation/Library?id=get-started--html-css-javascript#link-surveyjs-resources) as well.

1. *(Optional)* <a href="https://github.com/parallax/jsPDF#readme" target="_blank">jsPDF</a>, <a href="https://github.com/JonatanPe/jsPDF-AutoTable#readme" target="_blank">jsPDF-AutoTable</a>, and <a href="https://sheetjs.com/" target="_blank">SheetJS</a>       
Third-party libraries that enable users to export survey results to a PDF or XLSX document. Export to CSV is supported out of the box.

1. <a href="https://tabulator.info/" target="_blank">Tabulator</a>      
A third-party library that renders interactive tables.

1. SurveyJS Dashboard plugin for Tabulator      
A library that integrates Survey Core with Tabulator.

The following code shows how to reference these libraries:

```html
<head>
    <!-- ... -->
    <!-- SurveyJS Form Library resources -->
    <script type="text/javascript" src="https://unpkg.com/survey-core/survey.core.min.js"></script>
    <!-- Uncomment the following lines if you also display the survey on the page -->
    <!-- <link href="https://unpkg.com/survey-core/defaultV2.min.css" type="text/css" rel="stylesheet"> -->
    <!-- <script type="text/javascript" src="https://unpkg.com/survey-js-ui/survey-js-ui.min.js"></script> -->

    <!-- jsPDF for export to PDF -->
    <script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/1.5.3/jspdf.min.js"></script>
    <script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/jspdf-autotable/3.0.10/jspdf.plugin.autotable.min.js"></script>
    
    <!-- SheetJS for export to Excel -->
    <script type="text/javascript" src="https://oss.sheetjs.com/sheetjs/xlsx.full.min.js"></script>

    <!-- Tabulator -->
    <link href="https://cdnjs.cloudflare.com/ajax/libs/tabulator/4.7.2/css/tabulator.min.css" rel="stylesheet">
    <script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/tabulator/4.7.2/js/tabulator.min.js"></script>

    <!-- SurveyJS plugin for Tabulator -->
    <link href="https://unpkg.com/survey-analytics/survey.analytics.tabulator.min.css" rel="stylesheet">
    <script src="https://unpkg.com/survey-analytics/survey.analytics.tabulator.min.js"></script>

    <script type="text/javascript" src="index.js"></script>
</head>
```

## Load Survey Results

When a respondent completes a survey, a JSON object with their answers is passed to the `SurveyModel`'s [`onComplete`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#onComplete) event handler. You should send this object to your server and store it with a specific survey ID (see [Handle Survey Completion](/form-library/documentation/get-started-html-css-javascript#handle-survey-completion)). A collection of such JSON objects is a data source for the Table View. This collection can be processed (sorted, filtered, paginated) on the server or on the client.

### Server-Side Data Processing

Server-side data processing enables the Table View to load survey results in small batches on demand and delegate sorting and filtering to the server. For this feature to work, the server must support these data operations. Refer to the following demo example on GitHub for information on how to configure the server and the client for this usage scenario:

[SurveyJS Dashboard: Table View - Server-Side Data Processing Demo Example](https://github.com/surveyjs/surveyjs-dashboard-table-view-nodejs-mongodb (linkStyle))

> The Table View allows users to save survey results as CSV, PDF, and XLSX documents. With server-side data processing, these documents contain only currently loaded data records. To export full datasets, you need to generate the documents on the server.

### Client-Side Data Processing

When data is processed on the client, the Table View loads the entire dataset at startup and applies sorting and filtering in a user's browser. This demands faster web connection and higher computing power but works smoother with small datasets.

To load survey results to the client, send the survey ID to your server and return an array of JSON objects with survey results:

```js
const SURVEY_ID = 1;

loadSurveyResults("https://your-web-service.com/" + SURVEY_ID)
    .then((surveyResults) => {
        // ...
        // Configure and render the Table View here
        // Refer to the help topics below
        // ...
    });

function loadSurveyResults (url) {
    return new Promise((resolve, reject) => {
        const request = new XMLHttpRequest();
        request.open('GET', url);
        request.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
        request.onload = () => {
            const response = request.response ? JSON.parse(request.response) : [];
            resolve(response);
        }
        request.onerror = () => {
            reject(request.statusText);
        }
        request.send();
    });
}
```

For demonstration purposes, this tutorial uses auto-generated survey results. The following code shows a survey model and a function that generates the survey results array:

```js
const surveyJson = {
    elements: [{
        name: "satisfaction-score",
        title: "How would you describe your experience with our product?",
        type: "radiogroup",
        choices: [
            { value: 5, text: "Fully satisfying" },
            { value: 4, text: "Generally satisfying" },
            { value: 3, text: "Neutral" },
            { value: 2, text: "Rather unsatisfying" },
            { value: 1, text: "Not satisfying at all" }
        ],
        isRequired: true
    }, {
        name: "nps-score",
        title: "On a scale of zero to ten, how likely are you to recommend our product to a friend or colleague?",
        type: "rating",
        rateMin: 0,
        rateMax: 10,
    }],
    showQuestionNumbers: "off",
    completedHtml: "Thank you for your feedback!",
};

function randomIntFromInterval(min: number, max: number): number {
    return Math.floor(Math.random() * (max - min + 1) + min);
}
function generateData() {
    const data = [];
    for (let index = 0; index < 100; index++) {
        const satisfactionScore = randomIntFromInterval(1, 5);
        const npsScore = satisfactionScore > 3 ? randomIntFromInterval(7, 10) : randomIntFromInterval(1, 6);
        data.push({
            "satisfaction-score": satisfactionScore,
            "nps-score": npsScore
        });
    }
    return data;
}
```

## Render the Table

The Table View is rendered by the `Tabulator` component. Pass the survey model and results to its constructor to instantiate it. Assign the produced instance to a constant that will be used later to render the component:

```js
const surveyJson = { /* ... */ };
function generateData() { /* ... */ }

const survey = new Survey.Model(surveyJson);

const surveyDataTable = new SurveyAnalyticsTabulator.Tabulator(
    survey,
    generateData()
);
```

The Table View should be rendered in a page element. Add this element to the page markup:

```html
<body>
    <div id="surveyDataTable"></div>
</body>
```

To render the Table View in the page element, call the `render(container)` method on the Tabulator instance you created previously:

```js
document.addEventListener("DOMContentLoaded", function() {
    surveyDataTable.render(document.getElementById("surveyDataTable"));
});
```

<details>
    <summary>View Full Code</summary>

```html
<!DOCTYPE html>
<html>
<head>
    <title>Table View: SurveyJS Dashboard</title>
    <meta charset="utf-8">
    <script type="text/javascript" src="https://unpkg.com/survey-core/survey.core.min.js"></script>

    <!-- jsPDF for export to PDF -->
    <script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/1.5.3/jspdf.min.js"></script>
    <script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/jspdf-autotable/3.0.10/jspdf.plugin.autotable.min.js"></script>
    
    <!-- SheetJS for export to Excel -->
    <script type="text/javascript" src="https://oss.sheetjs.com/sheetjs/xlsx.full.min.js"></script>

    <!-- Tabulator -->
    <link href="https://cdnjs.cloudflare.com/ajax/libs/tabulator/4.7.2/css/tabulator.min.css" rel="stylesheet">
    <script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/tabulator/4.7.2/js/tabulator.min.js"></script>

    <!-- SurveyJS plugin for Tabulator -->
    <link href="https://unpkg.com/survey-analytics/survey.analytics.tabulator.min.css" rel="stylesheet">
    <script src="https://unpkg.com/survey-analytics/survey.analytics.tabulator.min.js"></script>

    <script type="text/javascript" src="index.js"></script>
</head>
<body>
    <div id="surveyDataTable"></div>
</body>
</html>
```

```js
const surveyJson = {
    elements: [{
        name: "satisfaction-score",
        title: "How would you describe your experience with our product?",
        type: "radiogroup",
        choices: [
            { value: 5, text: "Fully satisfying" },
            { value: 4, text: "Generally satisfying" },
            { value: 3, text: "Neutral" },
            { value: 2, text: "Rather unsatisfying" },
            { value: 1, text: "Not satisfying at all" }
        ],
        isRequired: true
    }, {
        name: "nps-score",
        title: "On a scale of zero to ten, how likely are you to recommend our product to a friend or colleague?",
        type: "rating",
        rateMin: 0,
        rateMax: 10,
    }],
    showQuestionNumbers: "off",
    completedHtml: "Thank you for your feedback!",
};

const survey = new Survey.Model(surveyJson);

function randomIntFromInterval(min, max) {
    return Math.floor(Math.random() * (max - min + 1) + min);
}
function generateData() {
    const data = [];
    for (let index = 0; index < 100; index++) {
        const satisfactionScore = randomIntFromInterval(1, 5);
        const npsScore = satisfactionScore > 3 ? randomIntFromInterval(7, 10) : randomIntFromInterval(1, 6);
        data.push({
            "satisfaction-score": satisfactionScore,
            "nps-score": npsScore
        });
    }
    return data;
}

const surveyDataTable = new SurveyAnalyticsTabulator.Tabulator(
    survey,
    generateData()
);

document.addEventListener("DOMContentLoaded", function() {
    surveyDataTable.render(document.getElementById("surveyDataTable"));
});
```

</details>

[View Full Code on GitHub](https://github.com/surveyjs/code-examples/tree/main/dashboard-table-view/html-css-js (linkStyle))

## See Also

[Dashboard Demo Examples](/dashboard/examples/ (linkStyle))