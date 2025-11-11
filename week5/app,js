// app.js
class PageRankApp {
    constructor() {
        this.graph = null;
        this.pageRankScores = null;
        this.selectedNode = null;
        this.isComputing = false;
        this.sortBy = 'id'; // 'id', 'pagerank', 'friends'
        this.sortOrder = 'asc'; // 'asc', 'desc'
        
        this.initializeEventListeners();
        this.loadDefaultData();
    }

    initializeEventListeners() {
        document.getElementById('computeBtn').addEventListener('click', () => this.computePageRank());
        document.getElementById('resetBtn').addEventListener('click', () => this.resetGraph());
        
        // Add table header click listeners for sorting
        document.getElementById('nodeTable').addEventListener('click', (e) => {
            if (e.target.tagName === 'TH') {
                const column = e.target.textContent.toLowerCase();
                this.sortTable(column);
            }
        });
    }

    async loadDefaultData() {
        try {
            // Try to load the karate club dataset, if not available use default data
            let csvText;
            try {
                const response = await fetch('data/karate.csv');
                csvText = await response.text();
            } catch (error) {
                console.log('Using default graph data');
                csvText = this.getDefaultGraphData();
            }
            
            this.graph = this.parseCSVToGraph(csvText);
            this.renderGraph();
            this.updateTable();
        } catch (error) {
            console.error('Error loading data:', error);
            // Fallback to default data
            this.graph = this.createDefaultGraph();
            this.renderGraph();
            this.updateTable();
        }
    }

    getDefaultGraphData() {
        // Default Zachary's Karate Club dataset
        return `1,2
1,3
2,3
2,4
2,5
2,6
2,7
3,4
3,8
3,9
3,10
4,6
5,6
5,7
6,7
8,9
8,10
8,11
8,12
9,10
9,12
9,13
10,11
10,13
11,12
11,13
12,13
12,14
12,15
13,14
13,15
14,15
14,16
15,16
16,17
16,18
17,18
18,19
19,20
19,21
20,21
21,22
22,23
22,24
23,24
24,25
25,26
25,27
26,27
27,28
27,29
27,30
28,29
28,30
29,30
30,31
31,32
31,33
32,33
33,34`;
    }

    createDefaultGraph() {
        // Create a simple default graph if all else fails
        const nodes = [
            {id: 1}, {id: 2}, {id: 3}, {id: 4}, {id: 5},
            {id: 6}, {id: 7}, {id: 8}, {id: 9}, {id: 10}
        ];
        
        const edges = [
            {source: 1, target: 2}, {source: 1, target: 3}, {source: 2, target: 3},
            {source: 2, target: 4}, {source: 3, target: 4}, {source: 4, target: 5},
            {source: 5, target: 6}, {source: 6, target: 7}, {source: 7, target: 8},
            {source: 8, target: 9}, {source: 9, target: 10}, {source: 1, target: 10}
        ];

        const adjacencyList = {};
        nodes.forEach(node => {
            adjacencyList[node.id] = [];
        });

        edges.forEach(edge => {
            adjacencyList[edge.source].push(edge.target);
            adjacencyList[edge.target].push(edge.source);
        });

        return { nodes, edges, adjacencyList };
    }

    parseCSVToGraph(csvText) {
        const edges = [];
        const nodes = new Set();
        
        const lines = csvText.trim().split('\n');
        for (const line of lines) {
            const [source, target] = line.split(',').map(Number);
            if (!isNaN(source) && !isNaN(target)) {
                edges.push({ source, target });
                nodes.add(source);
                nodes.add(target);
            }
        }

        // Create adjacency list
        const adjacencyList = {};
        Array.from(nodes).sort((a, b) => a - b).forEach(node => {
            adjacencyList[node] = [];
        });

        edges.forEach(edge => {
            if (adjacencyList[edge.source] && !adjacencyList[edge.source].includes(edge.target)) {
                adjacencyList[edge.source].push(edge.target);
            }
            if (adjacencyList[edge.target] && !adjacencyList[edge.target].includes(edge.source)) {
                adjacencyList[edge.target].push(edge.source);
            }
        });

        return {
            nodes: Array.from(nodes).sort((a, b) => a - b).map(id => ({ id })),
            edges: edges,
            adjacencyList: adjacencyList
        };
    }

    async computePageRank() {
        if (this.isComputing) return;
        
        this.isComputing = true;
        const computeBtn = document.getElementById('computeBtn');
        computeBtn.disabled = true;
        computeBtn.textContent = 'Computing...';

        try {
            this.pageRankScores = await computePageRank(this.graph.adjacencyList, 50, 0.85);
            this.updateTable();
            this.renderGraph();
            
            // If a node was selected, update its recommendations
            if (this.selectedNode) {
                this.showNodeDetails(this.selectedNode);
            }
        } catch (error) {
            console.error('Error computing PageRank:', error);
            alert('Error computing PageRank scores: ' + error.message);
        } finally {
            this.isComputing = false;
            computeBtn.disabled = false;
            computeBtn.textContent = 'Compute PageRank';
        }
    }

    resetGraph() {
        this.pageRankScores = null;
        this.selectedNode = null;
        this.sortBy = 'id';
        this.sortOrder = 'asc';
        this.loadDefaultData();
        document.getElementById('nodeDetails').innerHTML = '<p>Click on a node in the graph or table to see details</p>';
    }

    sortTable(column) {
        // Determine what to sort by
        let sortKey;
        switch(column) {
            case 'id':
                sortKey = 'id';
                break;
            case 'pagerank':
                sortKey = 'pageRank';
                break;
            case 'friends':
                sortKey = 'friendCount';
                break;
            default:
                sortKey = 'id';
        }

        // Toggle order if same column clicked
        if (this.sortBy === sortKey) {
            this.sortOrder = this.sortOrder === 'asc' ? 'desc' : 'asc';
        } else {
            this.sortBy = sortKey;
            this.sortOrder = 'asc';
        }

        this.updateTable();
    }

    updateTable() {
        const tableBody = document.getElementById('tableBody');
        tableBody.innerHTML = '';

        // Prepare nodes with additional data for sorting
        const nodesWithData = this.graph.nodes.map(node => {
            const friends = this.graph.adjacencyList[node.id] || [];
            return {
                id: node.id,
                pageRank: this.pageRankScores ? this.pageRankScores[node.id] : 0,
                friendCount: friends.length,
                friends: friends
            };
        });

        // Sort nodes
        nodesWithData.sort((a, b) => {
            let aVal = a[this.sortBy];
            let bVal = b[this.sortBy];
            
            if (this.sortBy === 'friends') {
                aVal = a.friendCount;
                bVal = b.friendCount;
            }
            
            if (this.sortOrder === 'asc') {
                return aVal - bVal;
            } else {
                return bVal - aVal;
            }
        });

        // Update table headers to show sort state
        this.updateTableHeaders();

        // Populate table
        nodesWithData.forEach(nodeData => {
            const row = document.createElement('tr');
            row.dataset.nodeId = nodeData.id;
            
            const pageRank = this.pageRankScores ? this.pageRankScores[nodeData.id].toFixed(4) : 'N/A';
            
            row.innerHTML = `
                <td>${nodeData.id}</td>
                <td>${pageRank}</td>
                <td>${nodeData.friends.join(', ')}</td>
            `;
            
            row.addEventListener('click', () => this.selectNode(nodeData.id));
            
            if (this.selectedNode === nodeData.id) {
                row.classList.add('selected');
            }
            
            tableBody.appendChild(row);
        });
    }

    updateTableHeaders() {
        const headers = document.querySelectorAll('#nodeTable th');
        headers.forEach(header => {
            header.classList.remove('sort-asc', 'sort-desc');
            const column = header.textContent.toLowerCase();
            let sortKey;
            switch(column) {
                case 'id': sortKey = 'id'; break;
                case 'pagerank': sortKey = 'pageRank'; break;
                case 'friends': sortKey = 'friendCount'; break;
                default: sortKey = null;
            }
            
            if (sortKey === this.sortBy) {
                header.classList.add(this.sortOrder === 'asc' ? 'sort-asc' : 'sort-desc');
            }
        });
    }

    selectNode(nodeId) {
        this.selectedNode = nodeId;
        
        // Update table selection
        document.querySelectorAll('#tableBody tr').forEach(row => {
            row.classList.toggle('selected', parseInt(row.dataset.nodeId) === nodeId);
        });
        
        // Update graph selection
        if (window.graphRenderer) {
            window.graphRenderer.highlightNode(nodeId);
        }
        
        this.showNodeDetails(nodeId);
    }

    showNodeDetails(nodeId) {
        const nodeDetails = document.getElementById('nodeDetails');
        const friends = this.graph.adjacencyList[nodeId] || [];
        
        let recommendationsHtml = '';
        if (this.pageRankScores) {
            const recommendations = this.getRecommendations(nodeId);
            if (recommendations.length > 0) {
                recommendationsHtml = `
                    <div class="recommendations">
                        <h4>Recommended New Friends (Top 3 by PageRank):</h4>
                        ${recommendations.map(rec => `
                            <div class="recommendation-item">
                                Node ${rec.node} (PageRank: ${rec.score.toFixed(4)})
                                <button onclick="app.connectNodes(${nodeId}, ${rec.node})">Connect</button>
                            </div>
                        `).join('')}
                    </div>
                `;
            } else {
                recommendationsHtml = '<p>No new friend recommendations available (you might already be connected to everyone).</p>';
            }
        } else {
            recommendationsHtml = '<p>Compute PageRank to see friend recommendations.</p>';
        }
        
        const currentPageRank = this.pageRankScores ? this.pageRankScores[nodeId].toFixed(4) : 'N/A';
        
        nodeDetails.innerHTML = `
            <div class="node-info">
                <h4>Node ${nodeId}</h4>
                <p><strong>PageRank Score:</strong> ${currentPageRank}</p>
                <p><strong>Current Friends (${friends.length}):</strong> ${friends.length > 0 ? friends.join(', ') : 'None'}</p>
                ${recommendationsHtml}
            </div>
        `;
    }

    getRecommendations(nodeId) {
        if (!this.pageRankScores) return [];
        
        const currentFriends = new Set(this.graph.adjacencyList[nodeId] || []);
        currentFriends.add(parseInt(nodeId)); // Exclude self
        
        const recommendations = [];
        
        this.graph.nodes.forEach(node => {
            if (!currentFriends.has(node.id)) {
                recommendations.push({
                    node: node.id,
                    score: this.pageRankScores[node.id]
                });
            }
        });
        
        // Sort by PageRank score descending and take top 3
        return recommendations.sort((a, b) => b.score - a.score).slice(0, 3);
    }

    connectNodes(sourceId, targetId) {
        // Add edge to graph
        this.graph.edges.push({ source: sourceId, target: targetId });
        
        // Update adjacency list
        if (!this.graph.adjacencyList[sourceId].includes(targetId)) {
            this.graph.adjacencyList[sourceId].push(targetId);
        }
        if (!this.graph.adjacencyList[targetId].includes(sourceId)) {
            this.graph.adjacencyList[targetId].push(sourceId);
        }
        
        // Update visualization
        this.renderGraph();
        
        // Show success message
        alert(`Connected node ${sourceId} with node ${targetId}! PageRank will be recomputed.`);
        
        // Recompute PageRank
        this.computePageRank();
    }

    renderGraph() {
        if (window.graphRenderer) {
            window.graphRenderer.renderGraph(this.graph, this.pageRankScores);
        }
    }
}

// Initialize app when page loads
let app;
document.addEventListener('DOMContentLoaded', () => {
    app = new PageRankApp();
    window.app = app;
});
