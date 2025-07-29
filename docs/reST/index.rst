import pygame
import random
import sys

# PyGame dasturini ishga tushirish
pygame.init()

# Ekran o'lchamlari
SCREEN_WIDTH = 800
SCREEN_HEIGHT = 600
SCREEN = pygame.display.set_mode((SCREEN_WIDTH, SCREEN_HEIGHT))
pygame.display.set_caption("2D Arcade O'yin")

# Ranglar
BACKGROUND = (20, 30, 50)
PLAYER_COLOR = (50, 200, 150)
ENEMY_COLOR = (220, 60, 80)
TEXT_COLOR = (240, 240, 240)
UI_BG = (10, 15, 25)

# O'yin parametrlari
PLAYER_SPEED = 7
ENEMY_MIN_SPEED = 2
ENEMY_MAX_SPEED = 6
ENEMY_SPAWN_RATE = 45  # Qanchalik tez dushmanlar paydo bo'lishi
SCORE_INCREASE = 10  # Har bir dushmandan qochganda ball


class Player:
    """Foydalanuvchi tomonidan boshqariladigan o'yinchi klass"""

    def __init__(self):
        """O'yinchi xususiyatlarini ishga tushirish"""
        self.width = 50
        self.height = 30
        self.x = SCREEN_WIDTH // 2 - self.width // 2
        self.y = SCREEN_HEIGHT - 60
        self.speed = PLAYER_SPEED
        self.rect = pygame.Rect(self.x, self.y, self.width, self.height)

    def update(self, keys):
        """O'yinchining harakatini yangilash"""
        if keys[pygame.K_LEFT] and self.rect.left > 0:
            self.rect.x -= self.speed
        if keys[pygame.K_RIGHT] and self.rect.right < SCREEN_WIDTH:
            self.rect.x += self.speed

    def draw(self):
        """O'yinchini ekranga chizish"""
        pygame.draw.rect(SCREEN, PLAYER_COLOR, self.rect, border_radius=8)
        # O'yinchi uchun qo'shimcha dizayn
        pygame.draw.rect(SCREEN, (100, 250, 200), self.rect, 3, border_radius=8)


class Enemy:
    """O'yinchi uchun dushman bo'lgan klass"""

    def __init__(self):
        """Dushman xususiyatlarini ishga tushirish"""
        self.width = random.randint(30, 50)
        self.height = random.randint(30, 50)
        self.x = random.randint(0, SCREEN_WIDTH - self.width)
        self.y = -self.height
        self.speed = random.randint(ENEMY_MIN_SPEED, ENEMY_MAX_SPEED)
        self.rect = pygame.Rect(self.x, self.y, self.width, self.height)

    def update(self):
        """Dushmanning harakatini yangilash"""
        self.rect.y += self.speed
        # Dushman ekran pastidan chiqib ketganda True qaytaradi
        return self.rect.top > SCREEN_HEIGHT

    def draw(self):
        """Dushmanni ekranga chizish"""
        pygame.draw.rect(SCREEN, ENEMY_COLOR, self.rect, border_radius=6)
        # Dushman uchun qo'shimcha dizayn
        pygame.draw.rect(SCREEN, (250, 100, 120), self.rect, 2, border_radius=6)


def draw_ui(score, game_over):
    """O'yin interfeysini chizish (ball va o'yin holati)"""
    # UI uchun fon
    pygame.draw.rect(SCREEN, UI_BG, (0, 0, SCREEN_WIDTH, 50))
    pygame.draw.line(SCREEN, (40, 60, 100), (0, 50), (SCREEN_WIDTH, 50), 2)

    # Shriftlarni sozlash
    font = pygame.font.SysFont(None, 36)

    # Ballni chizish
    score_text = font.render(f"Ball: {score}", True, TEXT_COLOR)
    SCREEN.blit(score_text, (20, 10))

    # O'yin holati
    status_text = font.render("O'yin Tugadi!" if game_over else "O'yin Davom Etmoqda", True, TEXT_COLOR)
    SCREEN.blit(status_text, (SCREEN_WIDTH - status_text.get_width() - 20, 10))


def check_collision(player, enemies):
    """O'yinchi va dushmanlar o'rtasidagi to'qnashuvni tekshirish"""
    for enemy in enemies:
        if player.rect.colliderect(enemy.rect):
            return True
    return False


def main():
    """O'yinning asosiy tsikli"""
    clock = pygame.time.Clock()
    player = Player()
    enemies = []
    score = 0
    game_over = False
    enemy_timer = 0

    # O'yin asosiy tsikli
    while True:
        # Tizim hodisalarini boshqarish
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_r and game_over:
                    # O'yinni qayta boshlash
                    return main()
                if event.key == pygame.K_q:
                    pygame.quit()
                    sys.exit()

        if not game_over:
            # Klaviatura tugmalarini olish
            keys = pygame.key.get_pressed()
            player.update(keys)

            # Yangi dushmanlar yaratish
            enemy_timer += 1
            if enemy_timer > ENEMY_SPAWN_RATE:
                enemies.append(Enemy())
                enemy_timer = 0

            # Dushmanlarni yangilash va ball hisoblash
            for enemy in enemies[:]:
                if enemy.update():
                    enemies.remove(enemy)
                    score += SCORE_INCREASE

            # To'qnashuvni tekshirish
            game_over = check_collision(player, enemies)

        # Chizish jarayoni
        SCREEN.fill(BACKGROUND)

        # Dushmanlarni chizish
        for enemy in enemies:
            enemy.draw()

        # O'yinchini chizish
        player.draw()

        # UI elementlarini chizish
        draw_ui(score, game_over)

        # Agar o'yin tugagan bo'lsa
        if game_over:
            font = pygame.font.SysFont(None, 72)
            text = font.render("Qayta Boshlash uchun 'R' tugmasini bosing", True, TEXT_COLOR)
            text_rect = text.get_rect(center=(SCREEN_WIDTH // 2, SCREEN_HEIGHT // 2))
            SCREEN.blit(text, text_rect)

            small_font = pygame.font.SysFont(None, 36)
            quit_text = small_font.render("Chiqish uchun 'Q' tugmasini bosing", True, TEXT_COLOR)
            quit_rect = quit_text.get_rect(center=(SCREEN_WIDTH // 2, SCREEN_HEIGHT // 2 + 50))
            SCREEN.blit(quit_text, quit_rect)

        pygame.display.flip()
        clock.tick(60)


if __name__ == "__main__":
    main()
