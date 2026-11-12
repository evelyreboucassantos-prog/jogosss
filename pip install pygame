import pygame
import random
import math
import array

# Inicialização
pygame.init()
pygame.mixer.init(frequency=22050, size=-16, channels=1)

# Configurações da Tela
LARGURA, ALTURA = 600, 750
TELA = pygame.display.set_mode((LARGURA, ALTURA))
pygame.display.set_caption("Desafio Python CCM - Batalha Espacial")
RELOGIO = pygame.time.Clock()
FPS = 60

# Cores Retrô
PRETO = (15, 15, 25)
BRANCO = (245, 245, 245)
AZUL_NAVE = (60, 180, 255)
CIANO = (150, 240, 255)
VERMELHO = (255, 70, 70)
VERDE_ALIEN = (80, 230, 100)
ROXO_BOSS = (180, 70, 255)
AMARELO = (255, 220, 50)

# Gerador de Sons Retrô (8-bit) sem arquivos externos
def gerar_som(freq_inicial, freq_final, duracao):
    sample_rate = 22050
    n_samples = int(sample_rate * duracao)
    buf = array.array('h', [0] * n_samples)
    for i in range(n_samples):
        t = i / n_samples
        freq = freq_inicial + (freq_final - freq_inicial) * t
        val = 8000 if math.sin(2 * math.pi * freq * (i / sample_rate)) > 0 else -8000
        buf[i] = int(val * (1 - t)) # Fade out
    return pygame.mixer.Sound(buffer=buf)

SOM_TIRO = gerar_som(900, 300, 0.1)
SOM_EXPLOSAO = gerar_som(200, 40, 0.25)
SOM_DANO = gerar_som(150, 80, 0.3)

# Desenhar sprites em Pixel Art via matriz
def criar_sprite_pixel(matriz, paleta, escala=4):
    altura = len(matriz) * escala
    largura = len(matriz[0]) * escala
    superficie = pygame.Surface((largura, altura), pygame.SRCALPHA)
    for y, linha in enumerate(matriz):
        for x, pixel in enumerate(linha):
            if pixel in paleta:
                pygame.draw.rect(superficie, paleta[pixel], (x * escala, y * escala, escala, escala))
    return superficie

MATRIZ_JOGADOR = [
    "....B....",
    "...BCB...",
    "...BCB...",
    "..BWWWB..",
    ".BWWWWWB.",
    "BWWWRWWWB",
    "BW..W..WB",
    "B...A...B"
]

MATRIZ_ALIEN = [
    "..G...G..",
    "...G.G...",
    "..GGGGG..",
    ".G.GGG.G.",
    "GGGGGGGGG",
    "G.GGGGG.G",
    "G.G...G.G",
    "...G.G..."
]

MATRIZ_BOSS = [
    "...RRRRRRR...",
    "..RRRRRRRRR..",
    ".RRYYRRRYYRR.",
    "RRRRRRRRRRRRR",
    "RRRRRRRRRRRRR",
    ".R.RR.R.RR.R.",
    ".R....R....R."
]

PALETA = {
    'B': AZUL_NAVE, 'C': CIANO, 'W': BRANCO, 'R': VERMELHO,
    'A': AMARELO, 'G': VERDE_ALIEN, 'Y': AMARELO
}

IMG_JOGADOR = criar_sprite_pixel(MATRIZ_JOGADOR, PALETA, escala=5)
IMG_ALIEN = criar_sprite_pixel(MATRIZ_ALIEN, PALETA, escala=4)
IMG_BOSS = criar_sprite_pixel(MATRIZ_BOSS, {'R': ROXO_BOSS, 'Y': AMARELO}, escala=10)

# Classes do Jogo
class Jogador(pygame.sprite.Sprite):
    def __init__(self):
        super().__init__()
        self.image = IMG_JOGADOR
        self.rect = self.image.get_rect(center=(LARGURA // 2, ALTURA - 70))
        self.velocidade = 6
        self.vidas = 3
        self.cooldown = 0

    def update(self):
        teclas = pygame.key.get_pressed()
        if (teclas[pygame.K_LEFT] or teclas[pygame.K_a]) and self.rect.left > 10:
            self.rect.x -= self.velocidade
        if (teclas[pygame.K_RIGHT] or teclas[pygame.K_d]) and self.rect.right < LARGURA - 10:
            self.rect.x += self.velocidade
        if self.cooldown > 0:
            self.cooldown -= 1

    def atirar(self, grupo_tiros):
        if self.cooldown == 0:
            tiro = Tiro(self.rect.centerx, self.rect.top, -10, AMARELO)
            grupo_tiros.add(tiro)
            SOM_TIRO.play()
            self.cooldown = 15

class Tiro(pygame.sprite.Sprite):
    def __init__(self, x, y, vel_y, cor):
        super().__init__()
        self.image = pygame.Surface((6, 16))
        self.image.fill(cor)
        self.rect = self.image.get_rect(center=(x, y))
        self.vel_y = vel_y

    def update(self):
        self.rect.y += self.vel_y
        if self.rect.bottom < 0 or self.rect.top > ALTURA:
            self.kill()

class Inimigo(pygame.sprite.Sprite):
    def __init__(self, x, y, vel):
        super().__init__()
        self.image = IMG_ALIEN
        self.rect = self.image.get_rect(center=(x, y))
        self.vel_x = vel

    def update(self):
        self.rect.x += self.vel_x
        if self.rect.right >= LARGURA - 10 or self.rect.left <= 10:
            self.vel_x *= -1
            self.rect.y += 25

class Chefao(pygame.sprite.Sprite):
    def __init__(self, fase):
        super().__init__()
        self.image = IMG_BOSS
        self.rect = self.image.get_rect(center=(LARGURA // 2, 110))
        self.vel_x = 3 + fase
        self.vida_max = 20 + (fase * 10)
        self.vida = self.vida_max
        self.timer_tiro = 0

    def update(self, grupo_tiros_inimigos):
        self.rect.x += self.vel_x
        if self.rect.right >= LARGURA - 15 or self.rect.left <= 15:
            self.vel_x *= -1

        self.timer_tiro += 1
        if self.timer_tiro >= 45:
            self.timer_tiro = 0
            grupo_tiros_inimigos.add(Tiro(self.rect.centerx - 30, self.rect.bottom, 6, VERMELHO))
            grupo_tiros_inimigos.add(Tiro(self.rect.centerx + 30, self.rect.bottom, 6, VERMELHO))

# Função para criar ondas de inimigos
def criar_onda(fase):
    grupo = pygame.sprite.Group()
    linhas = min(2 + fase, 4)
    for linha in range(linhas):
        for col in range(6):
            x = 90 + col * 80
            y = 70 + linha * 55
            grupo.add(Inimigo(x, y, 2 + fase * 0.5))
    return grupo

# Loop Principal
def main():
    fonte = pygame.font.SysFont("Courier", 22, bold=True)
    fonte_grande = pygame.font.SysFont("Courier", 38, bold=True)

    estrelas = [[random.randint(0, LARGURA), random.randint(0, ALTURA), random.randint(1, 3)] for _ in range(50)]

    jogador = Jogador()
    grupo_jogador = pygame.sprite.GroupSingle(jogador)
    tiros_jogador = pygame.sprite.Group()
    tiros_inimigos = pygame.sprite.Group()

    fase = 1
    pontos = 0
    inimigos = criar_onda(fase)
    boss = pygame.sprite.GroupSingle()
    modo_boss = False
    game_over = False

    rodando = True
    while rodando:
        RELOGIO.tick(FPS)

        for evento in pygame.event.get():
            if evento.type == pygame.QUIT:
                rodando = False
            if evento.type == pygame.KEYDOWN:
                if evento.key == pygame.K_SPACE and not game_over:
                    jogador.atirar(tiros_jogador)
                if evento.key == pygame.K_r and game_over:
                    main()
                    return

        if not game_over:
            # Disparo contínuo ao segurar espaço
            if pygame.key.get_pressed()[pygame.K_SPACE]:
                jogador.atirar(tiros_jogador)

            grupo_jogador.update()
            tiros_jogador.update()
            tiros_inimigos.update()

            if not modo_boss:
                inimigos.update()
                # Inimigos atiram aleatoriamente
                if inimigos and random.random() < 0.02 + (fase * 0.005):
                    atirador = random.choice(inimigos.sprites())
                    tiros_inimigos.add(Tiro(atirador.rect.centerx, atirador.rect.bottom, 5, VERMELHO))

                # Colisão tiros com inimigos
                acertos = pygame.sprite.groupcollide(inimigos, tiros_jogador, True, True)
                for _ in acertos:
                    pontos += 100
                    SOM_EXPLOSAO.play()

                # Se derrotar todos os inimigos, chama o Chefão!
                if len(inimigos) == 0:
                    modo_boss = True
                    boss.add(Chefao(fase))
            else:
                boss.update(tiros_inimigos)
                if boss.sprite:
                    acertos_boss = pygame.sprite.spritecollide(boss.sprite, tiros_jogador, True)
                    for _ in acertos_boss:
                        boss.sprite.vida -= 1
                        pontos += 50
                        if boss.sprite.vida <= 0:
                            SOM_EXPLOSAO.play()
                            pontos += 1000
                            boss.empty()
                            modo_boss = False
                            fase += 1
                            inimigos = criar_onda(fase)
                            break

            # Colisões com o Jogador
            if pygame.sprite.spritecollide(jogador, tiros_inimigos, True) or \
               pygame.sprite.spritecollide(jogador, inimigos, True):
                jogador.vidas -= 1
                SOM_DANO.play()
                if jogador.vidas <= 0:
                    game_over = True

            for inimigo in inimigos:
                if inimigo.rect.bottom >= jogador.rect.top:
                    game_over = True

        # Desenho na tela
        TELA.fill(PRETO)

        # Fundo estrelado animado
        for estrela in estrelas:
            estrela[1] += estrela[2]
            if estrela[1] > ALTURA:
                estrela[1] = 0
                estrela[0] = random.randint(0, LARGURA)
            pygame.draw.rect(TELA, BRANCO, (estrela[0], estrela[1], 2, 2))

        grupo_jogador.draw(TELA)
        tiros_jogador.draw(TELA)
        tiros_inimigos.draw(TELA)
        inimigos.draw(TELA)
        boss.draw(TELA)

        # Barra de vida do Chefão
        if modo_boss and boss.sprite:
            largura_barra = 300
            proporcao = boss.sprite.vida / boss.sprite.vida_max
            pygame.draw.rect(TELA, VERMELHO, (LARGURA // 2 - 150, 45, largura_barra, 12))
            pygame.draw.rect(TELA, VERDE_ALIEN, (LARGURA // 2 - 150, 45, int(largura_barra * proporcao), 12))

        # HUD (Pontuação, Vidas e Fase)
        txt_pontos = fonte.render(f"PONTOS: {pontos}", True, AMARELO)
        txt_fase = fonte.render(f"FASE: {fase}", True, CIANO)
        txt_vidas = fonte.render(f"VIDAS: {'♥ ' * jogador.vidas}", True, VERMELHO)
        TELA.blit(txt_pontos, (15, 12))
        TELA.blit(txt_fase, (LARGURA // 2 - 45, 12))
        TELA.blit(txt_vidas, (LARGURA - 165, 12))

        if game_over:
            txt_go = fonte_grande.render("GAME OVER", True, VERMELHO)
            txt_restart = fonte.render("Pressione 'R' para jogar novamente", True, BRANCO)
            TELA.blit(txt_go, txt_go.get_rect(center=(LARGURA // 2, ALTURA // 2 - 20)))
            TELA.blit(txt_restart, txt_restart.get_rect(center=(LARGURA // 2, ALTURA // 2 + 30)))

        pygame.display.flip()

    pygame.quit()

if __name__ == "__main__":
    main()