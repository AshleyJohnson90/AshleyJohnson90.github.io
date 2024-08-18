# Design Structures and Algorithms Enhancement   

[artifact](https://github.com/AshleyJohnson90/text-based-game-original){:target="_blank" rel="noopener"}   
[enhancement](https://github.com/AshleyJohnson90/text-based-game-enhanced){:target="_blank" rel="noopener"}   

**Briefly describe the artifact.  What is it?  When was it created?**   
   
The artifact is a restaurant-themed text-based adventure game written in Python. The player navigates from location to location, acquiring items to complete the challenge using typed commands in the terminal. To win the game, the player must visit each location and add all 6 items to their inventory, they then must navigate to the final location, the Dining Room where they successfully serve the customer. If the player reaches the Dining Room before they collect all the items, they lose and have to restart the game. It was created as a final project in my very first coding class, IT140 Intro to Scripting.   
   
**Justify the inclusion of the artifact in your ePortfolio.  Why did you select this item?  What specific components showcase your skills and abilities in algorithms and data structure?  How was the artifact improved?**   
   
I selected this artifact because it was a perfect example of using a graph structure to store locations as nodes and connecting directions as edges. When I was first creating this project, I had not taken any classes on design structures or algorithms or really knew that there were more efficient ways to store and retrieve data than just basic dictionaries or lists. So with the knowledge and experience I acquired throughout the degree program, I created a more efficient data structure backbone to build upon as I continue to enhance the game and it becomes more complex.   
   
In the original artifact, the locations, directions, and items were all stored in one dictionary.   
   
```
rooms = {
    'Hallway': {'North': 'Stove', 'East': 'Hostess Counter', 'South': 'Kitchen', 'West': 'Bathroom'},
    'Bathroom': {'East': 'Hallway', 'item': 'Apron'},
    'Kitchen': {'North': 'Hallway', 'East': 'Sink Station', 'item': 'Serving Tray'},
    'Walk-in': {'West': 'Stove', 'item': 'Salad'},
    'Hostess Counter': {'North': 'Dining Room', 'West': 'Hallway', 'item': 'Order Pad'},
    'Stove': {'South': 'Hallway', 'East': 'Walk-in', 'item': 'Steak'},
    'Sink Station': {'West': 'Kitchen', 'item': 'Plate'},
    'Dining Room': ''
}
```
   
In my enhancement, I broke that up into a graph for connecting the locations and a dictionary for mapping the location to the item. This gave the code more modularity and made it simpler to edit and manipulate the way items could be retrieved.   

```
connections = [
    ('Hallway', 'south', 'Kitchen', 'north'),
    ('Kitchen', 'east', 'Bathroom', 'west'),
    ('Bathroom', 'east', 'Walk-In', 'west'),
    ('Walk-In', 'south', 'Hostess Counter', 'north'),
    ('Hostess Counter', 'east', 'Dining Room', 'west'),
    ('Hostess Counter', 'west', 'Stove', 'east'),
    ('Stove', 'north', 'Sink Station', 'south'),
    ('Sink Station', 'west', 'Dining Room', '')
]

# dictionary mapping the location to the item that's in it, Hallway and Dining Room do not contain items
self.items = {
    'Bathroom': 'Apron',
    'Kitchen': 'Serving Tray',
    'Walk-In': 'Salad',
    'Hostess Counter': 'Order Pad',
    'Stove': 'Steak',
    'Sink Station': 'Plate'
}
```   
   
To create the graph data structure, I specified the methods to add the locations and the connecting directions into the graph from the connections. The player needs to be able to go back and forth between the locations so the graph has to be undirected. That means a direction and its opposite direction have to be specified in the graph.   
   
```
# method to add locations(nodes) to the graph
def add_location(self, location_name):
    if location_name not in self.connected_locations:
        self.connected_locations[location_name] = {}

# method to add connections(edges) to connect locations to each other using an undirected graph
def add_connection(self, location_name1, direction, location_name2, opposite_direction):
    self.connected_locations[location_name1][direction] = location_name2
    self.connected_locations[location_name2][opposite_direction] = location_name1

#method to create the graph using the connections
def create_graph(self):
    for location in self.location_names:
        self.graph.add_location(location)

    connections = [
        ('Hallway', 'south', 'Kitchen', 'north'),
        ('Kitchen', 'east', 'Bathroom', 'west'),
        ('Bathroom', 'east', 'Walk-In', 'west'),
        ('Walk-In', 'south', 'Hostess Counter', 'north'),
        ('Hostess Counter', 'east', 'Dining Room', 'west'),
        ('Hostess Counter', 'west', 'Stove', 'east'),
        ('Stove', 'north', 'Sink Station', 'south'),
        ('Sink Station', 'west', 'Dining Room', '')
    ]
    for location1, direction, location2, opposite_direction in connections:
        self.graph.add_connection(location1, direction, location2, opposite_direction)
```   
   
I was then able to create a cheat using a depth-first search algorithm to start in the Hallway, visit every room at least once, and end in the Dining Room. **The player will type "cheat" at the very first prompt to access the cheat**   
   
```
# depth first search to find the shortest path to visit all the locations and end in the Dining Room
def shortest_path(self, start_location, end_location):
    NUMBER_OF_LOCATIONS = 8
    path_directions = []
    visited = set()

    # every location that is visited needs to be stored in a set so the algorithm knows it has visited the location and not
    # an infinite loop through the locations
    def dfs(current_location):
        if current_location in visited:
            return False

        visited.add(current_location)

        # if the location is the end location and all 8 locations have been visited, return True that there is a path
        if current_location == end_location and len(visited) == NUMBER_OF_LOCATIONS:
            return True
        # if the connected location is not in visited, add the direction to the path directions list
        for direction, connected_location in self.connected_locations[current_location].items():
            if connected_location not in visited:
                path_directions.append(direction)
                if dfs(connected_location):
                    return True
                path_directions.pop()

        visited.remove(current_location)
        return False

    # if the depth first search finds a path, return the list of path directions or none if no valid path found
    if dfs(start_location):
        return path_directions
    else:
        return None
```   
   
**Did you meet the course outcomes you planned to meet with this enhancement in Module One?  Do you have any updates to your outcome-coverage plans?**   

I did meet the course outcomes I planned to meet with this enhancement. I wanted to meet the outcomes:   
1. Design and evaluate computing solutions that solve a given problem using algorithmic principles and computer science practices and standards appropriate to its solution while managing the trade-offs involved in design choices.
2. Develop a security mindset that anticipates adversarial exploits in software architecture and designs to expose potential vulnerabilities, mitigate design flaws, and ensure privacy and enhanced security of data and resources
   
The problem with the existing code is that it was sufficient for a basic game with only a few locations to move between, but it would not hold up when adding more complexity. My solution was to update the data structure to a graph and add a depth-first search algorithm cheat to traverse the graph and show the player the directions of the shortest path that visits every room and ends in the Dining Room to win the game. The player will now be able to move quickly between rooms and as more rooms are added, will have a secret cheat that will show them how to easily win the game.   

In the original artifact, if the player typed "get" and any words after that, it would take the item even if they did not tell it to, and if you typed any first word and a direction it would move in that direction.   
```
        if len(direction) >= 2 and direction[1] in rooms[current_room].keys():
            current_room = move_rooms(current_room, direction[1], rooms)
            continue
        # getting the item in the room
        elif len(direction[0]) >= 2 and direction[0] == 'Get':
            print('You grab the {}'.format(rooms[current_room]['item']))
            print('-----------------------------------------------------------------------------------------')
            grab_item(current_room, direction, rooms, inventory)
            continue
        # what happens if player enters something other than direction in room dictionary or action
        else:
            print('Ooopsies you cannot do that, try another way')
            print('-----------------------------------------------------------------------------------------')
            continue

```
   
I increased security in my enhancement by being more specific with my input validation. The player must use specific phrases to move or take the item.   
   
```
    # player must enter 'go' and then a direction to make a valid move
            if action[0] == 'go' and len(action) == 2:
                direction = action[1]
                self.move_locations(direction)
            # player must enter 'take item' to put item in inventory
            elif action[0] == 'take' and action[1] == 'item':
                self.grab_item()
            # player can enter 'cheat' to use the dfs to find the path to win, must use cheat as the very first entry
            elif action[0] == 'cheat':
                result = self.graph.shortest_path('Hallway', 'Dining Room')
                if result:
                    path_directions = result
                    path = " -> ".join(path_directions)
                    print(f'Path to visit all locations and end in the Dining Room: {path}\n'
                          '-----------------------------------------------------------------------------------------')
                else:
                    print('No valid path found ending in the Dining Room.')
            else:
                print('Invalid command. Use "go <direction>" or "take item".\n'
```   
   
**Reflect on the process of enhancing and modifying the artifact.  What did you learn as you were creating it and improving it?  What challenges did you face?**   

While improving the artifact, I learned about the different types of graph data structures and the algorithms that can be used to traverse them. I had never worked with graphs before this enhancement so everything about them was new to me. I learned the difference between a directed and undirected graph which was useful when creating the connections between the locations. It was a key feature that the player could go back and forth between locations so learning about the undirected graph was fundamental. I also learned about implementing the graph using an adjacency list or adjacency matrix. Because I needed to list the directions between the nodes, I could narrow the implementation down to an adjacency list. I ran into challenges with creating the depth-first search algorithm to find the shortest path to win the game. I was running into difficulties figuring out how to return the directions, rather than the nodes visited. It wouldn’t help the player to know the next room to visit if they didn’t know which direction that room was.
