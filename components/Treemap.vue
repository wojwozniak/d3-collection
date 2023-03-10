<template>
    <div id="link-container">
        <a id="videogames-link"></a>
        |
        <a id="movies-link"></a>
        |
        <a id="kickstarter-link"></a>
    </div>
    <h1 id="title">Video Game Sales</h1>
    <h3 id="description">Top 100 Most Sold Video Games by Platform</h3>
    <div id="visHolder" class="treemap"><svg id="chart" height="700" width="960"></svg></div>
</template>

<script>
import * as d3 from 'd3'
import styles from '~/assets/css/Treemap.scss'

export default {
    name: 'Treemap',
    data() {
        return {
            styles
        }
    },
    mounted() {
        this.renderD3();
    },
    methods: {
        renderD3() {
            const KICKSTARTER_PLEDGES = "https://cdn.freecodecamp.org/testable-projects-fcc/data/tree_map/kickstarter-funding-data.json";
            const MOVIE_SALES = "https://cdn.freecodecamp.org/testable-projects-fcc/data/tree_map/movie-data.json";
            const VIDEOGAME_SALES = "https://cdn.freecodecamp.org/testable-projects-fcc/data/tree_map/video-game-sales-data.json";

            // Data selection buttons
            const videogames_link = document.getElementById("videogames-link");
            const movies_link = document.getElementById("movies-link");
            const kickstarter_link = document.getElementById("kickstarter-link");

            // Title and subtitle selection
            const title = document.getElementById("title");
            const subtitle = document.getElementById("description");

            // Received data
            let kickstarter = {};
            let movies = {};
            let videogames = {};

            // Declare dataset
            let dataset = [];

            // Get data from API
            Promise.all([d3.json(KICKSTARTER_PLEDGES), d3.json(MOVIE_SALES), d3.json(VIDEOGAME_SALES)])
                .then((data) => {
                    kickstarter = data[0];
                    movies = data[1];
                    videogames = data[2];
                    main();
                    
                })
                .catch((err) => console.log(err));


            // Adding text to buttons
            const renderButtons = () => {
                videogames_link.textContent = "Video Games Chart";
                movies_link.textContent = "Movie Chart";
                kickstarter_link.textContent = "Kickstarter Chart";
            }

            // Color scale - code snippet from original FCC project
            const fader = (color) => d3.interpolateRgb(color, '#fff')(0.2);
            const color =  d3.scaleOrdinal().range(
                [
                '#1f77b4',
                '#aec7e8',
                '#ff7f0e',
                '#ffbb78',
                '#2ca02c',
                '#98df8a',
                '#d62728',
                '#ff9896',
                '#9467bd',
                '#c5b0d5',
                '#8c564b',
                '#c49c94',
                '#e377c2',
                '#f7b6d2',
                '#7f7f7f',
                '#c7c7c7',
                '#bcbd22',
                '#dbdb8d',
                '#17becf',
                '#9edae5'
                ].map(fader)
            );

            // Chart constants
            const treemap = d3.treemap().size([960, 500]);
            const svg = d3.select("svg");

            //Adding tooltip
            let tooltip = d3
                .select("body")
                .append("div")
                .attr("class", "tooltip")
                .attr("id", "tooltip")
                .style("opacity", 0);

            // D3
            function renderChart(data) {
                // Clear data before rendering new chart
                svg.selectAll("*").remove();

                // Root of diagram
                const root = d3
                    .hierarchy(data)
                    .eachBefore((d) => {
                        d.data.id = (d.parent ? d.parent.data.id + '.' : '') + d.data.name;
                    })
                    .sum((d) => d.value)
                    .sort((a, b) => {
                        return b.height - a.height || b.value - a.value;
                    });
                treemap(root);
                
                // Render group (which will contain rect and text)
                const visual = svg
                    .selectAll("g")
                    .data([0])
                    .enter()
                    .append("g")
                    .attr("id", "chart-items");
                
                const cell = visual
                    .selectAll('g')
                    .data(root.leaves())
                    .enter()
                    .append('g')
                    .attr('class', 'group')
                    .attr('transform', (d) => {
                    return 'translate(' + d.x0 + ',' + d.y0 + ')';
                    });
                
                // Render rect element
                cell
                    .append('rect')
                    .attr('id', (d) => d.data.id)
                    .attr('class', 'tile')
                    .attr('width', (d) => d.x1 - d.x0)
                    .attr('height', (d) => d.y1 - d.y0)
                    .attr('data-name', (d) => d.data.name)
                    .attr('data-category', (d) => d.data.category)
                    .attr('data-value', (d) => d.data.value)
                    .attr('fill', (d) => color(d.data.category));
                
                // Render text
                cell
                    .append('text')
                    .attr('class', 'label')
                    .selectAll('tspan')
                    //Regex splitting text to make it fit, snippet from main FreeCodeCamp project
                    .data((d) => d.data.name.split(/(?=[A-Z][^A-Z])/g))
                    .enter()
                    .append('tspan')
                    .attr('x', 4)
                    .attr('y', (d, i) => 13 + i * 10)
                    .text((d) => d);
                
                //Display tooltip
                cell
                    .on("mouseover", (d, dataPoint) => {
                        tooltip
                            .transition()
                            .duration(200)
                            .style("opacity", 0.9)
                            .attr("data-value", dataPoint.data.value);
                        tooltip
                            .html(() => {
                                if (dataset === videogames) {
                                    return `${dataPoint.data.name}<br>Category: ${dataPoint.data.category}<br><b>${dataPoint.data.value} Million Copies Sold</b>`
                                } else return `${dataPoint.data.name}<br>Category: ${dataPoint.data.category}<br><b>$${dataPoint.data.value}</b>`;
                            })
                            .style("left", d.pageX + 10 + "px")
                            .style("top", d.pageY + 10 + "px");
                    })
                    .on("mouseout", () => {
                        tooltip
                            .transition()
                            .duration(400)
                            .style("opacity", 0);
                    });
                const renderLegend = () => {
                    // Legend constants
                    const legendScale = 30;

                    // Get categories for legend
                    const categories = data.children.map((i) => i.name);
                    console.log(categories);

                    //Create groups for legend
                    const legend = d3
                        .select("#chart")
                        .append("g")
                        .attr("id", "legend")
                        .selectAll("g")
                        .data(categories)
                        .enter()
                        .append("g")
                        .attr("class", "legend-itemo");
                    
                    // Render rects
                    const legendRects = d3
                        .selectAll(".legend-itemo")
                        .append("rect")
                        .attr("height", legendScale)
                        .attr("width", legendScale)
                        .style("fill", (d) => color(d))
                        .attr("y", (d, i) => {
                            return 562 + i % 4 * (legendScale + 5);
                        })
                        .attr("x", (d, i) => {
                            return 160 + Math.floor(i / 4) * (legendScale + 145);
                        })
                        .attr("class", "legend-item");
                    
                    // Render labels for rects
                    const legendLabels = d3
                        .selectAll(".legend-itemo")
                        .append("text")
                        .attr("y", (d, i) => {
                            return 562 + 2 * legendScale / 3 + i % 4 * (legendScale + 5);
                        })
                        .attr("x", (d, i) => {
                            return 156 + 4 * legendScale / 3 + Math.floor(i / 4) * (legendScale + 145);
                        })
                        .text((d) => d);
                    
                    // Render legend title
                    const legendTitle = visual
                        .append("text")
                        .attr("id", "legend-title")
                        .text("Legend")
                        .attr("x", 50)
                        .attr("y", 580);
                    

                }
                renderLegend();
            }

            // Main function
            function main() {
                // Default dataset
                dataset = videogames;

                //Render buttons and default chart
                renderButtons();
                renderChart(dataset);
            }

            // Click listeners - setting data to selected by user, displaying correct title and subtitle and then calling renderChart function
            videogames_link.onclick = () => {
                dataset = videogames;
                title.textContent = "Video Game Sales";
                subtitle.textContent = "Top 100 Most Sold Video Games by Platform"
                renderChart(dataset);
            }
            movies_link.onclick = () => {
                dataset = movies;
                title.textContent = "Movies Box Office";
                subtitle.textContent = "Top 100 Movies with Highest Gross Income"
                renderChart(dataset);
            }
            kickstarter_link.onclick = () => {
                dataset = kickstarter;
                title.textContent = "Kickstarter Top Funded Projects";
                subtitle.textContent = "Top 100 Most Funded Kickstarter Projects"
                renderChart(dataset);
            }
        }

    }
}
</script>