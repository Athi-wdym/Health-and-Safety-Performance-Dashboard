# Health-and-Safety-Performance-Dashboard
The HSE Dashboard for tracking Near miss
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>HSE Performance Dashboard</title>
    <script src="https://cdn.plot.ly/plotly-latest.min.js"></script>
</head>
<body>
    <h2>Upload JSON File:</h2>
    <input type="file" id="fileInput">
    <div id="dashboard">
        <div id="nearMissTrend"></div>
        <div id="injurySeverity"></div>
        <div id="mostFrequentNearMiss"></div>
        <div id="injuredVsNotInjured"></div>
        <div id="heatmap"></div>
    </div>

    <script>
        document.getElementById('fileInput').addEventListener('change', function(event) {
            const file = event.target.files[0];
            if (!file) return;
            
            const reader = new FileReader();
            reader.onload = function(e) {
                const jsonData = JSON.parse(e.target.result);
                generateDashboard(jsonData);
            };
            reader.readAsText(file);
        });

        function generateDashboard(data) {
            // Extracting necessary fields
            const dates = data.map(item => item.Date);
            const nearMisses = data.map(item => item["Near miss"]);
            const injuryLevels = data.map(item => item["Injury level"]);
            const injured = data.map(item => item["Injured"] ? 1 : 0);
            const notInjured = data.map(item => item["Not Injured"] ? 1 : 0);
            
            // Line Chart - Near Miss Trend
            Plotly.newPlot('nearMissTrend', [{
                x: dates,
                y: nearMisses.map(() => 1),
                type: 'scatter',
                mode: 'lines+markers',
                name: 'Near Misses'
            }], { title: 'Near Miss Incidents Over Time' });

            // Pie Chart - Injury Severity Breakdown
            const severityCounts = {};
            injuryLevels.forEach(level => severityCounts[level] = (severityCounts[level] || 0) + 1);
            Plotly.newPlot('injurySeverity', [{
                labels: Object.keys(severityCounts),
                values: Object.values(severityCounts),
                type: 'pie'
            }], { title: 'Injury Severity Breakdown' });
            
            // Bar Chart - Most Frequent Near Miss Categories
            const nearMissCounts = {};
            nearMisses.forEach(miss => nearMissCounts[miss] = (nearMissCounts[miss] || 0) + 1);
            Plotly.newPlot('mostFrequentNearMiss', [{
                x: Object.keys(nearMissCounts),
                y: Object.values(nearMissCounts),
                type: 'bar'
            }], { title: 'Most Frequent Near Miss Categories' });
            
            // Stacked Bar Chart - Injured vs Not Injured
            Plotly.newPlot('injuredVsNotInjured', [{
                x: dates,
                y: injured,
                type: 'bar',
                name: 'Injured'
            }, {
                x: dates,
                y: notInjured,
                type: 'bar',
                name: 'Not Injured'
            }], { title: 'Injured vs Not Injured Cases', barmode: 'stack' });
            
            // Heatmap - Near Miss Severity Over Time
            const severityMapping = { "Low": 1, "Medium": 2, "High": 3 };
            const severityValues = injuryLevels.map(level => severityMapping[level] || 0);
            Plotly.newPlot('heatmap', [{
                z: [severityValues],
                x: dates,
                y: ['Severity'],
                type: 'heatmap'
            }], { title: 'Near Miss Heatmap by Date & Severity' });
        }
    </script>
</body>
</html>
