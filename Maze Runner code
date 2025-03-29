import API
import sys
from collections import deque

def log(string):
    sys.stderr.write("{}\n".format(string))
    sys.stderr.flush()

class MazeSolver:
    def __init__(self):
        self.width = API.mazeWidth()
        self.height = API.mazeHeight()
        self.current_x = 0
        self.current_y = 0
        self.direction = 0  # 0:North, 1:East, 2:South, 3:West
        self.visited = set()
        self.loop_tracker = {}  # Tracks visit counts to prevent loops
        self.graph = {}    # Store connectivity information
        self.explore_path = []
        self.optimal_path = []
        self.phase = "explore"

    def get_center(self):
        middle_x = self.width // 2
        middle_y = self.height // 2
        if self.width % 2 == 0:
            start_x = middle_x - 1
            end_x = middle_x
        else:
            start_x = middle_x
            end_x = middle_x

        if self.height % 2 == 0:
            start_y = middle_y - 1
            end_y = middle_y
        else:
            start_y = middle_y
            end_y = middle_y

        return start_x, start_y, end_x, end_y

    def is_in_center(self, x, y):
        start_x, start_y, end_x, end_y = self.get_center()
        return start_x <= x <= end_x and start_y <= y <= end_y

    def mark_center(self):
        start_x, start_y, end_x, end_y = self.get_center()
        for x in range(start_x, end_x + 1):
            for y in range(start_y, end_y + 1):
                API.setColor(x, y, "R")
                API.setText(x, y, "Center")

    def get_available_moves(self):
        moves = []
        current_pos = (self.current_x, self.current_y)
       
        if not API.wallFront():
            moves.append((0, self.direction))
        if not API.wallRight():
            moves.append((1, (self.direction + 1) % 4))
        if not API.wallLeft():
            moves.append((1, (self.direction - 1) % 4))
       
        # Update graph connectivity
        for _, direction in moves:
            dx, dy = [(0, 1), (1, 0), (0, -1), (-1, 0)][direction]
            next_pos = (self.current_x + dx, self.current_y + dy)
           
            if current_pos not in self.graph:
                self.graph[current_pos] = set()
            if next_pos not in self.graph:
                self.graph[next_pos] = set()
               
            self.graph[current_pos].add(next_pos)
            self.graph[next_pos].add(current_pos)
       
        return moves

    def manhattan_distance(self, x1, y1, x2, y2):
        return abs(x1 - x2) + abs(y1 - y2)

    def get_best_move(self, moves):
        best_move = None
        best_score = float('inf')

        for rotation_cost, move_direction in moves:
            dx, dy = [(0, 1), (1, 0), (0, -1), (-1, 0)][move_direction]
            next_x = self.current_x + dx
            next_y = self.current_y + dy

            if 0 <= next_x < self.width and 0 <= next_y < self.height:
                visit_count = self.loop_tracker.get((next_x, next_y), 0)
               
                if self.phase == "explore":
                    # During exploration, prioritize unvisited cells and path to center
                    start_x, start_y, end_x, end_y = self.get_center()
                    if self.is_in_center(next_x, next_y):
                        score = -100  # Highest priority for center cells
                    else:
                        distance = min(
                            self.manhattan_distance(next_x, next_y, start_x, start_y),
                            self.manhattan_distance(next_x, next_y, end_x, end_y)
                        )
                        score = distance + rotation_cost + (visit_count * 8)
               
                elif self.phase == "return":
                    # During return, prioritize path to start
                    distance = self.manhattan_distance(next_x, next_y, 0, 0)
                    path_bonus = -15 if (next_x, next_y) in self.explore_path else 0
                    score = distance + rotation_cost + (visit_count * 5) + path_bonus
               
                else:  # final phase
                    # During final run, follow optimal path
                    if (next_x, next_y) in self.optimal_path:
                        score = -1000 + rotation_cost  # Highest priority for optimal path
                    else:
                        score = float('inf')

                if score < best_score:
                    best_score = score
                    best_move = move_direction

        return best_move

    def find_shortest_path(self, start, target):
        queue = deque([(start, [start])])
        visited = {start}
       
        while queue:
            current, path = queue.popleft()
            if self.is_in_center(*current):
                return path
               
            for next_pos in self.graph.get(current, []):
                if next_pos not in visited:
                    visited.add(next_pos)
                    queue.append((next_pos, path + [next_pos]))
        return None

    def move_in_direction(self, target_direction):
        rotation_diff = (target_direction - self.direction) % 4
        if rotation_diff == 3:
            API.turnLeft()
        else:
            for _ in range(rotation_diff):
                API.turnRight()

        self.direction = target_direction
        try:
            API.moveForward()
            dx, dy = [(0, 1), (1, 0), (0, -1), (-1, 0)][self.direction]
            self.current_x += dx
            self.current_y += dy
            return True
        except API.MouseCrashedError:
            return False

    def solve(self):
        self.mark_center()
        API.setColor(0, 0, "G")
        API.setText(0, 0, "S")

        while True:
            current_position = (self.current_x, self.current_y)
           
            # Record position for path tracking
            if self.phase == "explore":
                self.explore_path.append(current_position)
               
                # Check if reached center during exploration
                if self.is_in_center(self.current_x, self.current_y):
                    log("Reached center! Computing shortest path...")
                    self.phase = "return"
                    # Find shortest path from start to center
                    self.optimal_path = self.find_shortest_path((0, 0), current_position)
                    self.visited.clear()
                    API.setColor(self.current_x, self.current_y, "B")
           
            # Check if returned to start
            elif self.phase == "return" and self.current_x == 0 and self.current_y == 0:
                log("Successfully returned to start! Starting final run...")
                self.phase = "final"
                self.visited.clear()
           
            # Check if reached center in final run
            elif self.phase == "final" and self.is_in_center(self.current_x, self.current_y):
                log("Completed final run to center!")
                break

            # Mark visited cells with appropriate color
            if current_position not in self.visited:
                self.visited.add(current_position)
                color = {"explore": "a", "return": "b", "final": "y"}[self.phase]
                API.setColor(self.current_x, self.current_y, color)

            # Update loop tracker
            self.loop_tracker[current_position] = self.loop_tracker.get(current_position, 0) + 1

            # Get and evaluate moves
            moves = self.get_available_moves()
            if not moves:
                API.turnRight()
                API.turnRight()
                self.direction = (self.direction + 2) % 4
                continue

            best_move = self.get_best_move(moves)
           
            if best_move is not None:
                if not self.move_in_direction(best_move):
                    API.turnRight()
                    API.turnRight()
                    self.direction = (self.direction + 2) % 4
            else:
                API.turnRight()
                API.turnRight()
                self.direction = (self.direction + 2) % 4

def main():
    log("Starting maze solver...")
    solver = MazeSolver()
    solver.solve()

if __name__ == "__main__":
    main()
