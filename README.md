# 🌐 Supply Chain Network Design with Linear Programming

A Python supply-chain network optimization project using **PuLP, geographic distance calculations, and linear programming** to evaluate distribution-center locations and customer allocations.

> 📓 **Full notebook:** [`notebooks/supply_chain_network_with_linear_programming.ipynb`](notebooks/supply_chain_network_with_linear_programming.ipynb)

## 🎯 Business Problem

The project addresses a strategic network-design question: **where should distribution centers be located and how should customer demand be allocated while balancing facility costs, transportation costs, capacity, and demand?**

The scenario is generated directly inside the notebook, so **no external dataset is required**.

---

# Supply Chain Network Design with Linear Programming

---

![linear opt4.gif](attachment:95073ae5-12fd-441a-aeab-5fc92fb4872e.gif)

---

# Introduction

---

Having recently delved into the fascinating world of **linear programming** in Python, I wanted to create a practical, hands-on example that not only solidifies my understanding but also serves as a valuable learning resource for other professionals approaching this powerful optimization technique. This article is designed as a test case, demonstrating how linear programming can be applied for supply chain network design.

For businesses, getting products from suppliers to customers efficiently and cost-effectively is paramount. One of the most fundamental strategic decisions in supply chain management is Supply Chain Network Design. This involves figuring out the optimal locations for your facilities, like warehouses, distribution centers, and factories.

Let's take an example where you're managing a growing company. Do you open one big warehouse, or several smaller ones? Where should they be located? How will they serve your customers most effectively and efficiently? These are the kinds of complex questions that Geographic Optimization can help answer.

This article will walk you through how we can use Python and a little bit of "smart math" (what we call optimization) to make these strategic location decisions. We'll simulate a simplified scenario and show how to find the best places for your distribution centers to serve customers efficiently.

It's important to note that this is an approach that needs to be explored and learned as first step in this field. Afterward, it will be necessary to understand and implement solutions that are much closer to real-world situations, which therefore involve repeated decisions over time whenever one of the variables in play changes. Always remembering to incorporate the uncertainty and variability that govern many systems today. Fortunately, there is a growing literature on the subject.

---

# The Problem: Where to Put Our Warehouses?

---

The challenge we're addressing is a classic problem known as the **Facility Location Problem**. We want to determine which new distribution centers (DCs) to open from a set of potential locations to best serve our customers. The goal is to minimize the total cost, which typically includes two main components:

* **Fixed Costs**: The cost of opening and operating a distribution center, regardless of how much product it handles (e.g., rent, utilities, initial setup).

* **Variable Costs**: The cost of shipping products from a DC to customers, which depends on the distance and volume.

**We need to decide**:

1.  Which potential distribution center locations should we choose to open?

2.  Once opened, which customers should each distribution center serve?

Opening more warehouses might reduce shipping costs (because they're closer to customers), but it increases fixed costs. Conversely, fewer warehouses mean lower fixed costs but potentially higher shipping costs. **Our optimization model will find the sweet spot!**

---

# Our Approach: Data-Driven Decisions with Optimization

---

To solve this, we'll follow a data-driven approach. We'll define:

1. Customer Locations and Demand: Where our customers are, and how much product they need.

2. Potential Distribution Center Locations: A list of places where we could open a warehouse.

3. Costs: The fixed cost of opening each potential warehouse and the variable cost of shipping goods from each warehouse to each customer.

We'll use a technique called Linear Programming. 
**Linear Programming** is a way to find the absolute best solution (like the lowest cost or highest profit) when everything you're dealing with can be described with straight lines and simple relationships. It's incredibly powerful for making decisions when you have many choices and constraints.

---

In the following code we will import the libraries we will use.

---

# Simulating Our Supply Chain Data

---

### haversine_distance function: 

This is a simple formula to calculate the approximate straight-line distance between two points on Earth using their latitude and longitude. This is how we'll estimate shipping distances and, therefore, shipping costs.

---

### Customer Data: 

We now create **50** imaginary customer locations with random latitude and longitude coordinates centered around Milan, Italy.

Each customer also has a demand, the amount of product they need.

---

The visualization part is crucial for business people to easily understand the strategic outcome. We use the **folium library**, which is excellent for creating interactive maps directly in Python.

Our customers are represented as the small blue circles as shown in the figure below.

---

### Potential DC Locations: 
We've identified 5 potential spots where we could build a distribution center. These have their own IDs and coordinates. 
The map below shows our 5 potential warehouses

---

### Costs (fixed_costs, shipping_cost_per_km_per_unit, variable_costs):

* **fixed_costs**: This is the annual cost of simply having a warehouse at each potential location, regardless of how much it ships.

* **shipping_cost_per_km_per_unit**: This is how much it costs to ship one unit of product for one kilometer.

* **variable_costs**: We calculate this for every possible connection: the cost of shipping all of a customer's demand from a specific potential DC. This cost is higher for further distances and higher customer demands.

---

### DC Capacities (dc_capacities): 
Each potential DC has a maximum amount of product it can handle per year. We can't serve unlimited customers from one small warehouse!

---

# Set Up the Optimization Model

---

This section is the "brain" of our solution, where we translate our business problem into a language that our optimizer (PuLP) can understand.


**Decision Variables (x and y)**: These are the "switches" that our model can flip.

x[dc_id][customer_id]: This is a binary switch (either 0 or 1). If x is 1, it means "Customer X will be served by Distribution Center Y." If it's 0, it means "Customer X will not be served by Distribution Center Y."

y[dc_id]: This is also a binary switch. If y is 1, it means "Distribution Center Z will be opened." If it's 0, it means "Distribution Center Z will not be opened."

**Objective Function** (model += fixed_cost_sum + variable_cost_sum, "Total Cost"):

This is the formula the model tries to make as small as possible.

It adds up two things:

1. The fixed costs for only the DCs that are chosen to be opened (because y will be 1 for them).

2. The variable shipping costs for only the customer-DC connections that are chosen (because x will be 1 for them).

**Constraints (model += ...)**:

These are the limitations our solution must follow.

1. "Each Customer Served Once": Ensures that every customer we have must be assigned to exactly one open distribution center. You can't leave a customer unserved, and they shouldn't be served by two different places for this problem type.

2. "Assign Only to Open DC": This is crucial. It makes sure that if a distribution center isn't selected to be opened (its y variable is 0), then no customers can be assigned to it.

3. "DC Capacity Limit": This ensures that the total demand from all customers assigned to an opened distribution center does not exceed that DC's maximum handling capacity.

---

# Running the Optimization and Visualizing Results

---

This part of the code is where the optimization happens!

**model.solve()**: This command tells PuLP to run the optimization. It crunches all the numbers, considers the objective, and applies all the constraints to find the absolute best combination of opened DCs and customer assignments that results in the Total Cost being minimized.

**pulp.LpStatus[model.status]**: This checks if the solver found a solution. Ideally, it will say "Optimal," meaning it found the best possible answer given our problem definition.

**pulp.value(model.objective)**: This retrieves the final, lowest total cost that the model found.

**opened_dcs**: We then look at our y variables to see which DCs were selected to be opened (where y[dc_id] is 1).

**customer_assignments**: We also check our x variables to see which customer is assigned to which opened DC.

**dc_served_demand**: This calculates how much total demand each opened DC is responsible for, allowing us to see if they're close to their capacity limits.

---

# Focusing On Business Implications

---

The output of our optimization run provides clear, actionable insights for your supply chain strategy. 
The solver successfully reached an **Optimal status**, indicating that it found the most cost-effective solution given all the defined parameters and constraints. 

This optimal configuration results in a **Total Optimized Cost of $1,066,391.00**, representing the lowest possible combined fixed and variable expenses for your network. **The model strategically identified that 'DC_North', 'DC_West', and 'DC_East' are the ideal distribution centers to open**. These locations, marked with green warehouse icons on the map, were chosen because their combined fixed costs and the variable shipping costs to serve their assigned customers resulted in the overall lowest total expense. They are strategically positioned to efficiently cover the customer demand in their respective regions.

Conversely, 'DC_South' and 'DC_Central', marked with red 'X' icons, were not selected. This decision implies that the fixed costs associated with opening and operating these facilities, when weighed against the potential savings in shipping costs they might offer, did not contribute to an overall lower total cost for the network. In essence, the model determined that the customer demand could be more economically served by the three chosen green DCs, even if it meant slightly longer shipping distances for some customers, because the fixed costs of 'DC_South' and 'DC_Central' outweighed those benefits. 

Furthermore, the results detail the Demand Served by Each Opened DC, showing how customer demand is efficiently allocated without exceeding any facility's capacity, ensuring smooth operations. These precise figures and selections are the direct outcome of the mathematical optimization, providing a robust, data-backed plan for your supply chain network.

---

# Conclusions: Understanding the Business Impact and What's Next

---

The results from this optimization model provide invaluable insights for strategic supply chain decisions:

* **Cost Savings**: The model gives you the absolute minimum total cost (fixed + variable) for your network, guiding you to the most economical setup.

* **Informed Location Decisions**: Instead of relying on intuition or simple distance calculations, you have a data-driven recommendation on where to place your facilities.

* **Efficient Resource Allocation**: You know exactly which customers each new (or existing) facility should serve, ensuring optimal utilization of capacity.

* **"What-If" Scenarios**: This model can be easily adapted to explore different scenarios:

-What if demand for a specific region increases dramatically?

-What if the fixed cost of opening a certain DC changes?

-What if a specific DC becomes unavailable (e.g., due to a disruption)?

By running these scenarios, businesses can build a more resilient and agile supply chain.

**Dynamic Decision-Making**: **It's crucial to understand that the optimal network design is not a static, one-time decision**. This optimization process should be repeated periodically, or whenever significant changes occur in your business environment. Factors such as the acquisition of new clients, shifts in their geographical locations, fluctuations in client demand, or changes in transportation and other operational costs necessitate a re-evaluation of your network.

**Future-Proofing through Scenario Planning**: **Effective network design must also consider future scenarios**. By incorporating projections for market growth, potential disruptions, or evolving customer expectations, businesses can make decisions that are robust and adaptable over time, moving beyond current conditions to anticipate future needs.

**Enriching the Model for Real-World Complexity**: While this example provides a strong foundation, it's a simplified representation of real-world complexities. For practical applications, **the decision variables and constraints would need to be significantly enriched, potentially moving beyond purely linear and deterministic assumptions to incorporate non-linear relationships (e.g., economies of scale that aren't perfectly linear) or stochastic elements (e.g., uncertain demand, variable lead times) to better reflect real-world variability**. This could involve considering multiple product categories, varying vehicle types and capacities, time windows for deliveries, labor availability, and more granular cost structures.

### Beyond This Example
While this example provides a solid foundation, real-world supply chain network design problems can be much more complex. Advanced models can incorporate:

* Multiple Product Types: Different products might have different storage or shipping requirements.

* Multi-Modal Transportation: Considering shipping by truck, rail, sea, or air, each with different costs and transit times.

* Time Windows & Service Level Agreements: Ensuring deliveries happen within specific timeframes.

* Dynamic Demand: Adjusting network design as demand patterns shift over time.

* Sustainability Goals: Adding objectives to minimize carbon emissions alongside costs.

* Uncertainty: Using techniques like Monte Carlo simulation to account for uncertain demand or costs.

By leveraging Python and optimization techniques, businesses can transform their supply chain network from a series of educated guesses into a strategically optimized, data-backed competitive advantage. It empowers decision-makers to build more efficient, resilient, and cost-effective supply chains for the future.

---

![linear opt5.gif](attachment:e4652b9e-4708-4499-868b-751a859470d7.gif)

---

## 🔄 Optimization Workflow

```text
Simulate Customers & Candidate DCs
              ↓
Calculate Geographic Distances
              ↓
Define Fixed / Variable / Shipping Costs
              ↓
Define DC Capacities
              ↓
Create Decision Variables
              ↓
Build Objective Function & Constraints
              ↓
Solve with PuLP
              ↓
Evaluate Selected DCs & Allocations
              ↓
Interpret Business Impact
```

## 🧮 Model Components

- Facility-opening decisions
- Customer-to-DC allocation
- Distribution-center capacities
- Customer demand
- Fixed facility costs
- Variable operating costs
- Distance-based shipping costs
- Haversine geographic distance

## 🛠️ Technologies

Python · PuLP · Pandas · NumPy · Folium · Matplotlib · Linear Programming · Jupyter

## 📂 Repository Structure

```text
supply_chain_network_linear_programming_project/
├── README.md
├── requirements.txt
├── .gitignore
└── notebooks/
    └── supply_chain_network_with_linear_programming.ipynb
```

## 🚀 Run Locally

```bash
git clone https://github.com/Ames0007/supply_chain_network_linear_programming_project.git
cd supply_chain_network_linear_programming_project
python -m venv .venv
source .venv/Scripts/activate
pip install -r requirements.txt
jupyter notebook notebooks/supply_chain_network_with_linear_programming.ipynb
```
