import pygame
import random
import os

# Initialize pygame
pygame.init()

# Define constants
WIDTH, HEIGHT = 800, 600
BACKGROUND_COLOR = (0, 0, 0)  # Black background
WHITE = (255, 255, 255)
RED = (255, 0, 0)
FPS = 60

# Set up the display
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Car Race Game")

# Load assets
def load_image(file_name):
    try:
        return pygame.image.load(os.path.join(file_name))
    except pygame.error as e:
        print(f"Error loading image: {file_name}")
        raise SystemExit(e)

car_img = load_image("car.png")
car_img = pygame.transform.scale(car_img, (50, 100))  # Resize the car image
bg_img = load_image("background.png")
bg_img = pygame.transform.scale(bg_img, (WIDTH, HEIGHT))  # Resize the background

# Define the Car class
class Car:
    def __init__(self, x, y):
        self.x = x
        self.y = y
        self.image = car_img
        self.rect = self.image.get_rect(center=(x, y))

    def draw(self):
        screen.blit(self.image, self.rect.topleft)

    def move(self, dx):
        """Move the car horizontally."""
        self.x += dx
        self.rect.x = self.x

        # Prevent the car from moving off-screen
        if self.rect.left < 0:
            self.rect.left = 0
        if self.rect.right > WIDTH:
            self.rect.right = WIDTH

# Define the Obstacle class
class Obstacle:
    def __init__(self):
        self.width = random.randint(50, 70)
        self.height = random.randint(50, 70)
        self.x = random.randint(100, WIDTH - 100)
        self.y = -self.height
        self.speed = random.randint(4, 8)
        self.color = RED

    def move(self):
        self.y += self.speed

    def draw(self):
        pygame.draw.rect(screen, self.color, (self.x, self.y, self.width, self.height))

    def reset(self):
        """Reset obstacle position."""
        self.y = -self.height
        self.x = random.randint(100, WIDTH - 100)

# Game loop
def game_loop():
    car = Car(WIDTH // 2, HEIGHT - 120)
    obstacles = [Obstacle() for _ in range(3)]  # Create 3 obstacles
    score = 0
    clock = pygame.time.Clock()
    run_game = True
    dx = 0

    while run_game:
        screen.fill(BACKGROUND_COLOR)  # Clear the screen with background color
        screen.blit(bg_img, (0, 0))  # Draw the background image

        # Event handling
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                run_game = False

        # Keyboard controls for car movement
        keys = pygame.key.get_pressed()
        if keys[pygame.K_LEFT]:
            dx = -5
        elif keys[pygame.K_RIGHT]:
            dx = 5
        else:
            dx = 0

        car.move(dx)

        # Move obstacles and check for collisions
        for obs in obstacles:
            obs.move()
            if obs.y > HEIGHT:  # If the obstacle has gone off the screen, reset it
                obs.reset()
                score += 1

            # Check collision with car
            car_rect = car.rect
            obstacle_rect = pygame.Rect(obs.x, obs.y, obs.width, obs.height)
            if car_rect.colliderect(obstacle_rect):
                print("Game Over!")
                print(f"Your Score: {score}")
                run_game = False

            # Draw the obstacle
            obs.draw()

        # Draw the car
        car.draw()

        # Display score
        font = pygame.font.SysFont("Arial", 30)
        score_text = font.render(f"Score: {score}", True, WHITE)
        screen.blit(score_text, (10, 10))

        # Update the screen
        pygame.display.update()

        # Set frame rate
        clock.tick(FPS)

    pygame.quit()

# Start the game loop
game_loop()
