import pygame
import random

# ゲーム画面サイズ
SCREEN_WIDTH = 300
SCREEN_HEIGHT = 600
BLOCK_SIZE = 30
COLUMN_COUNT = SCREEN_WIDTH // BLOCK_SIZE
ROW_COUNT = SCREEN_HEIGHT // BLOCK_SIZE

# 色定義
BLACK = (0, 0, 0)
GRAY = (128, 128, 128)
COLORS = [
    (0, 255, 255),  # I
    (0, 0, 255),    # J
    (255, 165, 0),  # L
    (255, 255, 0),  # O
    (0, 255, 0),    # S
    (128, 0, 128),  # T
    (255, 0, 0)     # Z
]

# テトリミノ定義
TETROMINOES = [
    [[1, 1, 1, 1]],                     # I
    [[1, 0, 0], [1, 1, 1]],             # J
    [[0, 0, 1], [1, 1, 1]],             # L
    [[1, 1], [1, 1]],                   # O
    [[0, 1, 1], [1, 1, 0]],             # S
    [[0, 1, 0], [1, 1, 1]],             # T
    [[1, 1, 0], [0, 1, 1]]              # Z
]

class Tetromino:
    def __init__(self, shape, color):
        self.shape = shape
        self.color = color
        self.x = COLUMN_COUNT // 2 - len(shape[0]) // 2
        self.y = 0

    def rotate(self):
        self.shape = [list(row) for row in zip(*self.shape[::-1])]

def check_collision(board, tetromino, dx, dy):
    for y, row in enumerate(tetromino.shape):
        for x, cell in enumerate(row):
            if cell:
                nx = tetromino.x + x + dx
                ny = tetromino.y + y + dy
                if nx < 0 or nx >= COLUMN_COUNT or ny >= ROW_COUNT or (ny >= 0 and board[ny][nx]):
                    return True
    return False

def merge_tetromino(board, tetromino):
    for y, row in enumerate(tetromino.shape):
        for x, cell in enumerate(row):
            if cell:
                board[tetromino.y + y][tetromino.x + x] = tetromino.color

def clear_lines(board):
    new_board = [row for row in board if any(cell == 0 for cell in row)]
    lines_cleared = ROW_COUNT - len(new_board)
    while len(new_board) < ROW_COUNT:
        new_board.insert(0, [0 for _ in range(COLUMN_COUNT)])
    return new_board, lines_cleared

def draw_board(screen, board, tetromino):
    screen.fill(BLACK)
    for y in range(ROW_COUNT):
        for x in range(COLUMN_COUNT):
            color = board[y][x] if board[y][x] else GRAY
            pygame.draw.rect(screen, color, (x*BLOCK_SIZE, y*BLOCK_SIZE, BLOCK_SIZE, BLOCK_SIZE), 0 if board[y][x] else 1)
    for y, row in enumerate(tetromino.shape):
        for x, cell in enumerate(row):
            if cell:
                pygame.draw.rect(screen, tetromino.color, ((tetromino.x + x) * BLOCK_SIZE, (tetromino.y + y) * BLOCK_SIZE, BLOCK_SIZE, BLOCK_SIZE))
    pygame.display.flip()

def main():
    pygame.init()
    screen = pygame.display.set_mode((SCREEN_WIDTH, SCREEN_HEIGHT))
    pygame.display.set_caption("Tetris")

    clock = pygame.time.Clock()
    board = [[0 for _ in range(COLUMN_COUNT)] for _ in range(ROW_COUNT)]
    tetromino = Tetromino(random.choice(TETROMINOES), random.choice(COLORS))
    fall_time = 0

    running = True
    while running:
        fall_time += clock.get_rawtime()
        clock.tick(30)

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False
            elif event.type == pygame.KEYDOWN:
                if event.key == pygame.K_LEFT and not check_collision(board, tetromino, -1, 0):
                    tetromino.x -= 1
                elif event.key == pygame.K_RIGHT and not check_collision(board, tetromino, 1, 0):
                    tetromino.x += 1
                elif event.key == pygame.K_DOWN and not check_collision(board, tetromino, 0, 1):
                    tetromino.y += 1
                elif event.key == pygame.K_UP:
                    original_shape = [row[:] for row in tetromino.shape]
                    tetromino.rotate()
                    if check_collision(board, tetromino, 0, 0):
                        tetromino.shape = original_shape

        if fall_time > 500:
            if not check_collision(board, tetromino, 0, 1):
                tetromino.y += 1
            else:
                merge_tetromino(board, tetromino)
                board, _ = clear_lines(board)
                tetromino = Tetromino(random.choice(TETROMINOES), random.choice(COLORS))
                if check_collision(board, tetromino, 0, 0):
                    running = False
            fall_time = 0

        draw_board(screen, board, tetromino)

    pygame.quit()

if __name__ == "__main__":
    main()
