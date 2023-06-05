import pygame
import random

# Initialize Pygame
pygame.init()

# Set the screen dimensions
WIDTH = 640
HEIGHT = 480

# Set the maze dimensions and cell size
maze_width = 21
maze_height = 15
cell_size = WIDTH // maze_width

# Set the player and ghost image paths
player_image_path = "rabbit.png"
ghost_images_paths = ["ghost.png"]
price_image_path = "carrot.png"

# Load the images
player_image = pygame.transform.scale(pygame.image.load(player_image_path), (cell_size, cell_size))
ghost_images = [pygame.transform.scale(pygame.image.load(path), (cell_size, cell_size)) for path in ghost_images_paths]
price_image = pygame.transform.scale(pygame.image.load(price_image_path), (cell_size, cell_size))

# Create the screen
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Pac-Rabbit")

# Game variables
clock = pygame.time.Clock()
game_over = False
player_pos = [WIDTH // 2, HEIGHT // 2]
player_speed = 5
score = 0
level = 1
lives = 3
kill_timer = 0

# Maze variables
maze = [
    "####################",
    "#..................#",
    "#.###.###..###.###.#",
    "#.#...#......#...#.#",
    "#.#.###.####.###.#.#",
    "#.#.#..........#.#.#",
    "#.....####.####.....",
    "#.#.#..........#.#.#",
    "#.#.###.####.###.#.#",
    "#.#...#......#...#.#",
    "#.###.###..###.###.#",
    "#..................#",
    "####################"
]
maze_width = len(maze[0])
maze_height = len(maze)
cell_size = min(WIDTH // maze_width, HEIGHT // maze_height)

# Dot variables
dots = []
for row in range(maze_height):
    for col in range(maze_width):
        if maze[row][col] == '.':
            dots.append((col * cell_size + cell_size // 2, row * cell_size + cell_size // 2))
dots_collected = []

# Price variables
price = (random.choice(dots))
price_collected = False
price_timer = 0  # Timer to track when to respawn the price
price_respawn_delay = 5 * 10  # 5 seconds (10 frames per second)

# Ghost variables
ghosts = [
    {
        "position": [random.randint(0, WIDTH - cell_size), random.randint(0, HEIGHT - cell_size)],
        "direction": random.choice(["up", "down", "left", "right"]),
        "speed": 1
    }
    for _ in range(6)
]

# Game loop
while not game_over:
    clock.tick(10)  # Set the game speed to 10 frames per second

    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            game_over = True

        # Player movement
        if event.type == pygame.KEYDOWN:
            if event.key == pygame.K_LEFT:
                player_pos[0] -= player_speed
            elif event.key == pygame.K_RIGHT:
                player_pos[0] += player_speed
            elif event.key == pygame.K_UP:
                player_pos[1] -= player_speed
            elif event.key == pygame.K_DOWN:
                player_pos[1] += player_speed

    # Collision detection
    if player_pos[0] < 0 or player_pos[0] >= WIDTH or player_pos[1] < 0 or player_pos[1] >= HEIGHT:
        game_over = True

    # Update game logic

    # Update ghost positions
    for ghost in ghosts:
        if ghost["direction"] == "up":
            ghost["position"][1] -= ghost["speed"]
        elif ghost["direction"] == "down":
            ghost["position"][1] += ghost["speed"]
        elif ghost["direction"] == "left":
            ghost["position"][0] -= ghost["speed"]
        elif ghost["direction"] == "right":
            ghost["position"][0] += ghost["speed"]

        # Ghost collision detection
        if (
            player_pos[0] - cell_size // 2 < ghost["position"][0] < player_pos[0] + cell_size // 2 and
            player_pos[1] - cell_size // 2 < ghost["position"][1] < player_pos[1] + cell_size // 2
        ):
            if kill_timer == 0:
                lives -= 1
                if lives == 0:
                    game_over = True
                else:
                    player_pos = [WIDTH // 2, HEIGHT // 2]
            else:
                ghosts.remove(ghost)

        # Ghost wall obstacle collision detection
        if (
                ghost["position"][0] < 0 or ghost["position"][0] >= WIDTH or
                ghost["position"][1] < 0 or ghost["position"][1] >= HEIGHT or
                maze[int(ghost["position"][1] // cell_size)][int(ghost["position"][0] // cell_size)] == '#'
        ):
            # Ghost got stuck or hit a wall, change direction
            ghost["direction"] = random.choice(["up", "down", "left", "right"])

    # Player wall obstacle collision detection
    if (
        maze[int(player_pos[1] // cell_size)][int(player_pos[0] // cell_size)] == '#' or
        player_pos[0] < 0 or player_pos[0] >= WIDTH or
        player_pos[1] < 0 or player_pos[1] >= HEIGHT
    ):
        # Player against wall obstacle, force player to go in the opposite direction
        if event.key == pygame.K_LEFT:
            player_pos[0] += player_speed
        elif event.key == pygame.K_RIGHT:
            player_pos[0] -= player_speed
        elif event.key == pygame.K_UP:
            player_pos[1] += player_speed
        elif event.key == pygame.K_DOWN:
            player_pos[1] -= player_speed

    # Slow down the ghosts' movement
    for ghost in ghosts:
        if ghost["position"][0] % cell_size == 0 and ghost["position"][1] % cell_size == 0:
            ghost["speed"] = random.choice([1, 2])

    # Check collision with dots
    for dot in dots:
        if (
            player_pos[0] - cell_size // 2 < dot[0] < player_pos[0] + cell_size // 2 and
            player_pos[1] - cell_size // 2 < dot[1] < player_pos[1] + cell_size // 2
        ):
            dots.remove(dot)
            dots_collected.append(dot)
            score += 1
            if score % 20 == 0:
                level += 1

    # Check collision with the price
    if (
        player_pos[0] - cell_size // 2 < price[0] < player_pos[0] + cell_size // 2 and
        player_pos[1] - cell_size // 2 < price[1] < player_pos[1] + cell_size // 2
    ):
        price_collected = True
        kill_timer = 80  # Set the kill timer to 8 seconds (10 frames per second * 80 frames)
        price_timer = price_respawn_delay  # Start the timer to respawn the price

    # Update price respawn timer
    if price_timer > 0:
        price_timer -= 1

        # Respawn the price if the timer reaches 0
        if price_timer == 0:
            price = random.choice(dots)
            price_collected = False

    # Update game graphics
    screen.fill((0, 0, 0))  # Clear the screen

    # Draw the maze
    for row in range(maze_height):
        for col in range(maze_width):
            if maze[row][col] == '#':
                pygame.draw.rect(screen, (255, 255, 255), (col * cell_size, row * cell_size, cell_size, cell_size))

    # Draw the dots
    for dot in dots:
        pygame.draw.circle(screen, (255, 255, 0), dot, 2)

    # Draw the price (carrot)
    if not price_collected:
        pygame.draw.circle(screen, (255, 165, 0), price, 8)

    # Draw the ghosts
    for ghost in ghosts:
        pygame.draw.rect(screen, (255, 0, 0), (ghost["position"][0], ghost["position"][1], cell_size, cell_size))

    # Draw the player
    pygame.draw.circle(screen, (255, 255, 255), player_pos, 10)

    # Draw the score and level
    font = pygame.font.SysFont("Arial", 18)
    score_text = font.render("Score: " + str(score), True, (255, 255, 255))
    level_text = font.render("Level: " + str(level), True, (255, 255, 255))
    screen.blit(score_text, (10, 10))
    screen.blit(level_text, (10, 30))

    # Draw lives
    lives_text = font.render("Lives: " + str(lives), True, (255, 255, 255))
    screen.blit(lives_text, (10, 80))

    pygame.display.update()

    # Decrease the kill timer
    if kill_timer > 0:
        kill_timer -= 1

# Game over screen
screen.fill((0, 0, 0))
font = pygame.font.SysFont("Arial", 48)
game_over_text = font.render("Game Over", True, (255, 0, 0))
screen.blit(game_over_text, (WIDTH // 2 - game_over_text.get_width() // 2, HEIGHT // 2 - game_over_text.get_height() // 2))
pygame.display.update()
pygame.time.delay(2000)

# Quit the game
pygame.quit()
