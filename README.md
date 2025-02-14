# snake 
import tkinter as tk
import random

# Constants
GAME_WIDTH = 700
GAME_HEIGHT = 500
SQUARE_SIZE = 20
SNAKE_COLOR = "green"
FOOD_COLOR = "red"
BACKGROUND_COLOR = "black"

class SnakeGame:
    def __init__(self, root):
        self.root = root
        self.root.title("Snake Game")
        self.root.resizable(False, False)

        # Canvas setup
        self.canvas = tk.Canvas(root, width=GAME_WIDTH, height=GAME_HEIGHT, bg=BACKGROUND_COLOR)
        self.canvas.pack()

        # Game variables
        self.snake = [(100, 100)]  # Initial snake coordinates
        self.food = None
        self.direction = "Right"
        self.running = True
        self.score = 0
        self.direction_changed = False  # To prevent multiple changes in one tick

        # Bind keys
        self.root.bind("<Up>", lambda event: self.change_direction("Up"))
        self.root.bind("<Down>", lambda event: self.change_direction("Down"))
        self.root.bind("<Left>", lambda event: self.change_direction("Left"))
        self.root.bind("<Right>", lambda event: self.change_direction("Right"))

        # Initialize game
        self.create_food()
        self.update()
        self.root.mainloop()

    def change_direction(self, new_direction):
        """Change the direction of the snake."""
        opposites = {"Up": "Down", "Down": "Up", "Left": "Right", "Right": "Left"}
        if not self.direction_changed and new_direction != opposites.get(self.direction, ""):
            self.direction = new_direction
            self.direction_changed = True  # Prevent additional changes in the same tick

    def create_food(self):
        """Place food at a random location, avoiding the snake's body."""
        while True:
            x = random.randint(0, (GAME_WIDTH // SQUARE_SIZE) - 1) * SQUARE_SIZE
            y = random.randint(0, (GAME_HEIGHT // SQUARE_SIZE) - 1) * SQUARE_SIZE
            if (x, y) not in self.snake:  # Ensure food is not placed on the snake
                self.food = (x, y)
                self.canvas.create_rectangle(x, y, x + SQUARE_SIZE, y + SQUARE_SIZE, fill=FOOD_COLOR, tag="food")
                break

    def update(self):
        """Update the game state."""
        if not self.running:
            return

        # Move the snake
        x, y = self.snake[-1]
        if self.direction == "Up":
            y -= SQUARE_SIZE
        elif self.direction == "Down":
            y += SQUARE_SIZE
        elif self.direction == "Left":
            x -= SQUARE_SIZE
        elif self.direction == "Right":
            x += SQUARE_SIZE

        new_head = (x, y)

        # Check for collisions
        if (
            x < 0 or x >= GAME_WIDTH or
            y < 0 or y >= GAME_HEIGHT or
            new_head in self.snake
        ):
            self.game_over()
            return

        # Check if food is eaten
        if new_head == self.food:
            self.snake.append(new_head)
            self.canvas.delete("food")
            self.create_food()
            self.drawsnake()
            self.score += 1
        else:
            # Move snake forward
            self.snake.append(new_head)
            self.snake.pop(0)

        # Update the canvas
        self.canvas.delete("snake")
        for segment in self.snake:
            x, y = segment
            self.canvas.create_rectangle(x, y, x + SQUARE_SIZE, y + SQUARE_SIZE, fill=SNAKE_COLOR, tag="snake")

        self.direction_changed = False  # Allow direction change after the update
        # Schedule the next update
        self.root.after(100, self.update)

    def game_over(self):
        """End the game."""
        self.running = False
        self.canvas.create_text(
            GAME_WIDTH // 2,
            GAME_HEIGHT // 2,
            fill="white",
            font="Times 20 bold",
            text=f"Game Over! Score: {self.score}"
        )

# Create and run the game
if __name__== "__main__":
    root = tk.Tk()
    game = SnakeGame(root)
