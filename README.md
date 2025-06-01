# glowing-giggle
公共经济学大作业
import pygame
import sys
import math
import random
from pygame.locals import *

# 初始化pygame
pygame.init()
pygame.font.init()

# 屏幕设置
WIDTH, HEIGHT = 1000, 700
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("弹性市场大亨 - 经济学模拟游戏")

# 颜色定义
BACKGROUND = (25, 30, 45)
PANEL_BG = (40, 45, 65)
HEADER_BG = (30, 85, 150)
TEXT_COLOR = (220, 220, 240)
HIGHLIGHT = (70, 150, 220)
BUTTON_BG = (50, 120, 180)
BUTTON_HOVER = (70, 160, 220)
LOW_ELASTIC = (120, 220, 150)  # 低弹性商品颜色
MED_ELASTIC = (220, 200, 120)  # 中弹性商品颜色
HIGH_ELASTIC = (220, 150, 120)  # 高弹性商品颜色
PROFIT_COLOR = (120, 220, 150)
LOSS_COLOR = (220, 120, 120)
GRID_COLOR = (70, 75, 95)

# 字体
title_font = pygame.font.SysFont("simhei", 36, bold=True)
header_font = pygame.font.SysFont("simhei", 28, bold=True)
main_font = pygame.font.SysFont("simhei", 22)
small_font = pygame.font.SysFont("simhei", 18)
tiny_font = pygame.font.SysFont("simhei", 16)

# 商品类
class Product:
    def __init__(self, name, category, base_price, base_demand, elasticity, cost, min_price, max_price):
        self.name = name
        self.category = category  # "low", "medium", "high"
        self.base_price = base_price
        self.price = base_price
        self.base_demand = base_demand
        self.demand = base_demand
        self.elasticity = elasticity  # 价格弹性系数
        self.cost = cost  # 成本
        self.min_price = min_price
        self.max_price = max_price
        self.price_history = [base_price]
        self.demand_history = [base_demand]
        self.profit_history = [(base_price - cost) * base_demand]
        
    def update_demand(self):
        # 根据弹性系数更新需求
        price_change = (self.price - self.base_price) / self.base_price
        demand_change = self.elasticity * price_change
        new_demand = max(10, min(self.base_demand * (1 + demand_change), self.base_demand * 3))
        self.demand = int(new_demand)
        self.demand_history.append(self.demand)
        
    def set_price(self, new_price):
        # 设置价格并更新需求
        self.price = max(self.min_price, min(self.max_price, new_price))
        self.price_history.append(self.price)
        self.update_demand()
        self.profit_history.append((self.price - self.cost) * self.demand)
        
    def get_profit(self):
        return (self.price - self.cost) * self.demand

# 事件卡牌类
class EventCard:
    def __init__(self, title, description, effect):
        self.title = title
        self.description = description
        self.effect = effect  # 影响函数

# 游戏类
class ElasticMarketGame:
    def __init__(self):
        self.products = [
            Product("食盐", "low", 2.5, 980, -0.2, 1.2, 2.0, 3.0),
            Product("感冒药", "low", 15.0, 450, -0.3, 7.0, 12.0, 20.0),
            Product("瓶装水", "low", 1.5, 1200, -0.1, 0.5, 1.0, 2.0),
            Product("牛仔裤", "medium", 199.0, 150, -1.5, 120.0, 150.0, 300.0),
            Product("智能手机", "medium", 799.0, 120, -1.2, 500.0, 600.0, 1000.0),
            Product("电视机", "medium", 2999.0, 80, -1.8, 2000.0, 2500.0, 4000.0),
            Product("海岛游套餐", "high", 1500.0, 35, -3.0, 800.0, 800.0, 2000.0),
            Product("名牌手表", "high", 8999.0, 15, -2.5, 4000.0, 7000.0, 12000.0),
            Product("钻石项链", "high", 12999.0, 8, -3.5, 6000.0, 10000.0, 20000.0)
        ]
        
        self.week = 1
        self.total_profit = 0
        self.profit_history = []
        self.profit_margin_history = []
        self.selected_product = 0
        self.adjustments_left = 3
        self.event_active = None
        self.event_cards = self.create_event_cards()
        self.game_state = "playing"  # "playing", "win", "lose"
        self.consecutive_weeks = 0
        self.price_adjustment = 0
        
    def create_event_cards(self):
        cards = [
            EventCard("公共卫生事件", "流感爆发，药品需求激增！", self.health_event),
            EventCard("网红带货", "社交媒体网红推荐了你的产品！", self.influencer_event),
            EventCard("替代品上市", "市场出现竞争产品！", self.competitor_event),
            EventCard("节日促销季", "节日到来，消费需求增加！", self.holiday_event),
            EventCard("经济危机", "经济下滑，消费者购买力下降！", self.economic_crisis)
        ]
        return cards
    
    def health_event(self):
        for product in self.products:
            if "药" in product.name:
                product.base_demand = int(product.base_demand * 1.8)
                product.demand = int(product.demand * 1.8)
    
    def influencer_event(self):
        # 随机选择一个商品
        idx = random.randint(3, 8)  # 选择中高弹性商品
        product = self.products[idx]
        # 降低弹性（需求变得更不敏感）
        product.elasticity *= 0.7
        # 增加需求
        product.base_demand = int(product.base_demand * 1.5)
        product.demand = int(product.demand * 1.5)
        return f"网红推荐了{product.name}！"
    
    def competitor_event(self):
        # 随机选择一个商品
        idx = random.randint(3, 8)  # 选择中高弹性商品
        product = self.products[idx]
        # 增加弹性（需求变得更敏感）
        product.elasticity *= 1.3
        # 减少需求
        product.base_demand = int(product.base_demand * 0.8)
        product.demand = int(product.demand * 0.8)
        return f"{product.name}出现新的竞争对手！"
    
    def holiday_event(self):
        for product in self.products[3:]:  # 中高弹性商品
            product.base_demand = int(product.base_demand * 1.4)
            product.demand = int(product.demand * 1.4)
        return "节日促销季到来！"
    
    def economic_crisis(self):
        for product in self.products[3:]:  # 中高弹性商品
            product.base_demand = int(product.base_demand * 0.7)
            product.demand = int(product.demand * 0.7)
        return "经济危机，消费降级！"
    
    def trigger_event(self):
        if self.week % 4 == 0:  # 每4周触发一次事件
            self.event_active = random.choice(self.event_cards)
            self.event_active.effect()
            return True
        return False
    
    def calculate_total_profit(self):
        total = 0
        for product in self.products:
            total += product.get_profit()
        self.total_profit = total
        self.profit_history.append(total)
        
        # 计算利润率
        total_revenue = sum(p.price * p.demand for p in self.products)
        total_cost = sum(p.cost * p.demand for p in self.products)
        margin = (total_revenue - total_cost) / total_revenue * 100 if total_revenue > 0 else 0
        self.profit_margin_history.append(margin)
        
        # 检查连续10周利润率>15%
        if margin > 15:
            self.consecutive_weeks += 1
            if self.consecutive_weeks >= 10:
                self.game_state = "win"
        else:
            self.consecutive_weeks = 0
        
        return total
    
    def next_week(self):
        self.week += 1
        self.adjustments_left = 3
        self.price_adjustment = 0
        
        # 随机波动基础需求
        for product in self.products:
            fluctuation = random.uniform(0.95, 1.05)
            product.base_demand = max(10, int(product.base_demand * fluctuation))
            product.update_demand()
        
        # 触发事件
        event_triggered = self.trigger_event()
        self.calculate_total_profit()
        
        return event_triggered
    
    def adjust_price(self, product_idx, adjustment):
        if self.adjustments_left <= 0:
            return False
        
        product = self.products[product_idx]
        new_price = product.price * (1 + adjustment/100)
        product.set_price(new_price)
        self.adjustments_left -= 1
        self.calculate_total_profit()
        return True

# 创建按钮类
class Button:
    def __init__(self, x, y, width, height, text, action=None):
        self.rect = pygame.Rect(x, y, width, height)
        self.text = text
        self.action = action
        self.hovered = False
        self.enabled = True
        
    def draw(self, surface):
        if not self.enabled:
            color = (100, 100, 120)
        else:
            color = BUTTON_HOVER if self.hovered else BUTTON_BG
            
        pygame.draw.rect(surface, color, self.rect, border_radius=8)
        pygame.draw.rect(surface, HIGHLIGHT, self.rect, 2, border_radius=8)
        
        text_surf = main_font.render(self.text, True, TEXT_COLOR)
        text_rect = text_surf.get_rect(center=self.rect.center)
        surface.blit(text_surf, text_rect)
        
    def check_hover(self, pos):
        self.hovered = self.rect.collidepoint(pos) and self.enabled
        
    def handle_event(self, event):
        if event.type == MOUSEBUTTONDOWN and event.button == 1:
            if self.hovered and self.action and self.enabled:
                return self.action()
        return False

# 创建滑块类
class Slider:
    def __init__(self, x, y, width, height, min_val=-30, max_val=30):
        self.rect = pygame.Rect(x, y, width, height)
        self.min_val = min_val
        self.max_val = max_val
        self.value = 0
        self.dragging = False
        self.handle_radius = 12
        self.handle_x = x + width // 2
        
    def draw(self, surface):
        # 绘制滑轨
        pygame.draw.rect(surface, PANEL_BG, self.rect, border_radius=4)
        pygame.draw.rect(surface, GRID_COLOR, self.rect, 1, border_radius=4)
        
        # 绘制中心线
        center_y = self.rect.y + self.rect.height // 2
        pygame.draw.line(surface, GRID_COLOR, (self.rect.x, center_y), 
                         (self.rect.x + self.rect.width, center_y), 2)
        
        # 绘制滑块
        handle_pos = (self.handle_x, center_y)
        pygame.draw.circle(surface, HIGHLIGHT, handle_pos, self.handle_radius)
        pygame.draw.circle(surface, TEXT_COLOR, handle_pos, self.handle_radius-3)
        
        # 绘制值
        value_text = small_font.render(f"{self.value:.1f}%", True, TEXT_COLOR)
        surface.blit(value_text, (self.rect.x + self.rect.width + 10, center_y - 10))
        
        # 绘制标签
        min_text = small_font.render(f"{self.min_val}%", True, TEXT_COLOR)
        max_text = small_font.render(f"{self.max_val}%", True, TEXT_COLOR)
        surface.blit(min_text, (self.rect.x - 30, center_y - 10))
        surface.blit(max_text, (self.rect.x + self.rect.width + 40, center_y - 10))
        
    def handle_event(self, event):
        if event.type == MOUSEBUTTONDOWN and event.button == 1:
            if self.is_over_handle(event.pos):
                self.dragging = True
                return True
        elif event.type == MOUSEBUTTONUP and event.button == 1:
            self.dragging = False
        elif event.type == MOUSEMOTION and self.dragging:
            self.update_value(event.pos[0])
            return True
        return False
    
    def is_over_handle(self, pos):
        center_y = self.rect.y + self.rect.height // 2
        distance = math.sqrt((pos[0] - self.handle_x)**2 + (pos[1] - center_y)**2)
        return distance <= self.handle_radius
    
    def update_value(self, mouse_x):
        mouse_x = max(self.rect.x, min(self.rect.x + self.rect.width, mouse_x))
        self.handle_x = mouse_x
        
        # 计算值 (-30 到 30)
        normalized = (mouse_x - self.rect.x) / self.rect.width
        self.value = self.min_val + normalized * (self.max_val - self.min_val)

# 绘制带圆角的矩形
def draw_rounded_rect(surface, rect, color, corner_radius):
    if corner_radius < 0:
        raise ValueError(f"Corner radius {corner_radius} must be >= 0")
    
    if corner_radius == 0:
        pygame.draw.rect(surface, color, rect)
    else:
        pygame.draw.rect(surface, color, rect, border_radius=corner_radius)

# 绘制仪表盘
def draw_dashboard(surface, game, x, y, width, height):
    # 绘制背景
    panel_rect = pygame.Rect(x, y, width, height)
    draw_rounded_rect(surface, panel_rect, PANEL_BG, 12)
    pygame.draw.rect(surface, GRID_COLOR, panel_rect, 2, border_radius=12)
    
    # 绘制标题
    title = header_font.render("商品价格仪表盘", True, TEXT_COLOR)
    surface.blit(title, (x + 20, y + 15))
    
    # 绘制表头
    headers = ["商品", "类别", "当前价格", "当前销量", "建议价格区间", "利润率", "操作"]
    col_width = width // len(headers)
    
    # 调整列宽，避免重叠
    col_widths = [
        int(width * 0.15),  # 商品
        int(width * 0.10),  # 类别
        int(width * 0.12),  # 当前价格
        int(width * 0.12),  # 当前销量
        int(width * 0.18),  # 建议价格区间
        int(width * 0.13),  # 利润率
        int(width * 0.15)   # 操作
    ]
    
    header_y = y + 60
    pygame.draw.rect(surface, HEADER_BG, (x, header_y, width, 40), border_radius=8)
    
    # 绘制表头文本
    cum_width = 0
    for i, header in enumerate(headers):
        header_text = main_font.render(header, True, TEXT_COLOR)
        header_rect = header_text.get_rect(center=(x + cum_width + col_widths[i] // 2, header_y + 20))
        surface.blit(header_text, header_rect)
        cum_width += col_widths[i]
    
    # 绘制商品行
    row_height = 40
    start_y = header_y + 50
    
    for idx, product in enumerate(game.products):
        row_y = start_y + idx * row_height
        
        # 交替行背景
        if idx % 2 == 0:
            pygame.draw.rect(surface, (50, 55, 75), (x, row_y, width, row_height))
        
        # 计算每列起始位置
        cum_col = 0
        
        # 绘制商品名称
        name_text = main_font.render(product.name, True, TEXT_COLOR)
        surface.blit(name_text, (x + 10, row_y + 12))
        cum_col += col_widths[0]
        
        # 绘制商品类别
        category_text = main_font.render(product.category, True, 
                                        LOW_ELASTIC if product.category == "low" 
                                        else MED_ELASTIC if product.category == "medium" 
                                        else HIGH_ELASTIC)
        surface.blit(category_text, (x + cum_col + 10, row_y + 12))
        cum_col += col_widths[1]
        
        # 绘制当前价格
        price_text = main_font.render(f"${product.price:.2f}", True, TEXT_COLOR)
        surface.blit(price_text, (x + cum_col + 10, row_y + 12))
        cum_col += col_widths[2]
        
        # 绘制当前销量
        demand_text = main_font.render(f"{product.demand}", True, TEXT_COLOR)
        surface.blit(demand_text, (x + cum_col + 10, row_y + 12))
        cum_col += col_widths[3]
        
        # 绘制建议价格区间
        range_text = main_font.render(f"${product.min_price:.2f}-${product.max_price:.2f}", True, TEXT_COLOR)
        surface.blit(range_text, (x + cum_col + 10, row_y + 12))
        cum_col += col_widths[4]
        
        # 绘制利润率
        profit_margin = ((product.price - product.cost) / product.price * 100) if product.price > 0 else 0
        margin_color = PROFIT_COLOR if profit_margin > 0 else LOSS_COLOR
        margin_text = small_font.render(f"{profit_margin:.1f}%", True, margin_color)  # 使用小字体避免重叠
        surface.blit(margin_text, (x + cum_col + 10, row_y + 12))
        cum_col += col_widths[5]
        
        # 绘制操作按钮
        button_rect = pygame.Rect(x + cum_col + 10, row_y + 5, 100, 30)
        pygame.draw.rect(surface, BUTTON_BG if idx != game.selected_product else BUTTON_HOVER, button_rect, border_radius=6)
        pygame.draw.rect(surface, HIGHLIGHT, button_rect, 2, border_radius=6)
        button_text = small_font.render("选择" if idx != game.selected_product else "已选择", True, TEXT_COLOR)
        surface.blit(button_text, (button_rect.centerx - button_text.get_width()//2, 
                                  button_rect.centery - button_text.get_height()//2))
    
    return start_y + len(game.products) * row_height + 20

# 绘制需求弹性公式
def draw_elasticity_formula(surface, x, y):
    panel_rect = pygame.Rect(x, y, 400, 120)
    draw_rounded_rect(surface, panel_rect, PANEL_BG, 12)
    pygame.draw.rect(surface, GRID_COLOR, panel_rect, 2, border_radius=12)
    
    title = header_font.render("需求价格弹性公式", True, HIGHLIGHT)
    surface.blit(title, (x + 20, y + 15))
    
    formula = main_font.render("E_d = (%ΔQ) / (%ΔP)", True, TEXT_COLOR)
    surface.blit(formula, (x + 50, y + 60))
    
    legend1 = small_font.render("E_d: 需求价格弹性系数", True, TEXT_COLOR)
    legend2 = small_font.render("%ΔQ: 需求量变化百分比", True, TEXT_COLOR)
    legend3 = small_font.render("%ΔP: 价格变化百分比", True, TEXT_COLOR)
    
    surface.blit(legend1, (x + 50, y + 85))
    surface.blit(legend2, (x + 50, y + 105))
    surface.blit(legend3, (x + 50, y + 125))
    
    # 弹性说明
    low_elastic = small_font.render("|E_d| < 1: 低弹性商品", True, LOW_ELASTIC)
    med_elastic = small_font.render("1 < |E_d| < 2: 中弹性商品", True, MED_ELASTIC)
    high_elastic = small_font.render("|E_d| > 2: 高弹性商品", True, HIGH_ELASTIC)
    
    surface.blit(low_elastic, (x + 220, y + 85))
    surface.blit(med_elastic, (x + 220, y + 105))
    surface.blit(high_elastic, (x + 220, y + 125))

# 绘制事件面板
def draw_event_panel(surface, game, x, y, width, height):
    panel_rect = pygame.Rect(x, y, width, height)
    draw_rounded_rect(surface, panel_rect, PANEL_BG, 12)
    pygame.draw.rect(surface, GRID_COLOR, panel_rect, 2, border_radius=12)
    
    title = header_font.render("市场事件", True, HIGHLIGHT)
    surface.blit(title, (x + 20, y + 15))
    
    if game.event_active:
        event_title = main_font.render(game.event_active.title, True, (220, 180, 80))
        event_desc = small_font.render(game.event_active.description, True, TEXT_COLOR)
        
        surface.blit(event_title, (x + width//2 - event_title.get_width()//2, y + 60))
        surface.blit(event_desc, (x + width//2 - event_desc.get_width()//2, y + 90))
    else:
        no_event = main_font.render("当前没有市场事件", True, TEXT_COLOR)
        surface.blit(no_event, (x + width//2 - no_event.get_width()//2, y + 75))
    
    # 绘制事件图标
    icon_size = 60
    icon_rect = pygame.Rect(x + width - icon_size - 20, y + 20, icon_size, icon_size)
    pygame.draw.rect(surface, HEADER_BG, icon_rect, border_radius=10)
    pygame.draw.rect(surface, HIGHLIGHT, icon_rect, 2, border_radius=10)
    
    event_icon = small_font.render("!", True, HIGHLIGHT)
    surface.blit(event_icon, (icon_rect.centerx - event_icon.get_width()//2, 
                            icon_rect.centery - event_icon.get_height()//2))

# 绘制游戏状态
def draw_game_status(surface, game, x, y, width, height):
    panel_rect = pygame.Rect(x, y, width, height)
    draw_rounded_rect(surface, panel_rect, PANEL_BG, 12)
    pygame.draw.rect(surface, GRID_COLOR, panel_rect, 2, border_radius=12)
    
    title = header_font.render("游戏状态", True, HIGHLIGHT)
    surface.blit(title, (x + 20, y + 15))
    
    # 周数
    week_text = main_font.render(f"第 {game.week} 周", True, TEXT_COLOR)
    surface.blit(week_text, (x + 30, y + 60))
    
    # 总利润
    profit_text = main_font.render(f"本周利润: ${game.total_profit:,.2f}", True, 
                                 PROFIT_COLOR if game.total_profit > 0 else LOSS_COLOR)
    surface.blit(profit_text, (x + 30, y + 90))
    
    # 利润率
    margin = game.profit_margin_history[-1] if game.profit_margin_history else 0
    margin_text = main_font.render(f"利润率: {margin:.1f}%", True, 
                                 PROFIT_COLOR if margin > 15 else TEXT_COLOR)
    surface.blit(margin_text, (x + 30, y + 120))
    
    # 连续周数
    consec_text = main_font.render(f"连续达标周数: {game.consecutive_weeks}/10", True, 
                                 PROFIT_COLOR if game.consecutive_weeks >= 7 else 
                                 (220, 200, 120) if game.consecutive_weeks >= 4 else TEXT_COLOR)
    surface.blit(consec_text, (x + 30, y + 150))
    
    # 剩余调整次数
    adjust_text = main_font.render(f"剩余价格调整次数: {game.adjustments_left}/3", True, 
                                 TEXT_COLOR if game.adjustments_left > 0 else (220, 120, 120))
    surface.blit(adjust_text, (x + 30, y + 180))
    
    # 胜利条件
    win_cond = small_font.render("目标: 连续10周利润率>15%", True, (180, 220, 180))
    surface.blit(win_cond, (x + 30, y + 220))

# 绘制价格调整面板
def draw_price_adjustment(surface, game, slider, x, y, width, height):
    panel_rect = pygame.Rect(x, y, width, height)
    draw_rounded_rect(surface, panel_rect, PANEL_BG, 12)
    pygame.draw.rect(surface, GRID_COLOR, panel_rect, 2, border_radius=12)
    
    title = header_font.render("价格调整", True, HIGHLIGHT)
    surface.blit(title, (x + 20, y + 15))
    
    if game.selected_product < len(game.products):
        product = game.products[game.selected_product]
        
        # 显示当前选中的商品
        product_text = main_font.render(f"当前选中: {product.name}", True, 
                                      LOW_ELASTIC if product.category == "low" 
                                      else MED_ELASTIC if product.category == "medium" 
                                      else HIGH_ELASTIC)
        surface.blit(product_text, (x + 30, y + 60))
        
        # 显示当前价格
        price_text = main_font.render(f"当前价格: ${product.price:.2f}", True, TEXT_COLOR)
        surface.blit(price_text, (x + 30, y + 90))
        
        # 显示建议价格区间
        range_text = main_font.render(f"建议区间: ${product.min_price:.2f}-${product.max_price:.2f}", True, TEXT_COLOR)
        surface.blit(range_text, (x + 30, y + 120))
        
        # 显示弹性系数
        elasticity_text = main_font.render(f"弹性系数: {abs(product.elasticity):.2f} ({product.category}弹性)", True, TEXT_COLOR)
        surface.blit(elasticity_text, (x + 30, y + 150))
        
        # 绘制滑块
        slider.rect.x = x + 30
        slider.rect.y = y + 190
        slider.draw(surface)
        
        # 预测需求变化
        if slider.value != 0:
            price_change = slider.value / 100
            demand_change = product.elasticity * price_change
            new_demand = max(10, product.demand * (1 + demand_change))
            
            demand_text = main_font.render(f"预测销量: {int(new_demand)} (变化: {demand_change*100:.1f}%)", True, 
                                         PROFIT_COLOR if new_demand > product.demand else LOSS_COLOR)
            surface.blit(demand_text, (x + 30, y + 230))
            
            # 预测利润
            new_price = product.price * (1 + price_change)
            new_profit = (new_price - product.cost) * new_demand
            current_profit = product.get_profit()
            profit_change = new_profit - current_profit
            
            profit_text = main_font.render(f"预测利润: ${new_profit:,.2f} (变化: {profit_change:,.2f})", True, 
                                         PROFIT_COLOR if profit_change > 0 else LOSS_COLOR)
            surface.blit(profit_text, (x + 30, y + 260))
    
    # 创建并绘制应用调整按钮
    apply_btn = Button(x + width//2 - 90, y + height - 45, 180, 40, 
                      "应用价格调整" if game.adjustments_left > 0 else "无调整次数",
                      action=lambda: game.adjust_price(game.selected_product, slider.value))
    apply_btn.enabled = game.adjustments_left > 0 and slider.value != 0
    apply_btn.draw(surface)
    
    return apply_btn

# 绘制游戏结束画面
def draw_game_over(surface, game):
    overlay = pygame.Surface((WIDTH, HEIGHT), pygame.SRCALPHA)
    overlay.fill((0, 0, 0, 200))
    surface.blit(overlay, (0, 0))
    
    panel_rect = pygame.Rect(WIDTH//2 - 250, HEIGHT//2 - 150, 500, 300)
    draw_rounded_rect(surface, panel_rect, (50, 60, 85), 20)
    pygame.draw.rect(surface, HIGHLIGHT, panel_rect, 3, border_radius=20)
    
    if game.game_state == "win":
        title = title_font.render("恭喜你获胜！", True, PROFIT_COLOR)
        subtitle = main_font.render("你成功连续10周保持利润率超过15%", True, TEXT_COLOR)
        achievement = main_font.render("成为一名真正的弹性市场大亨！", True, (220, 200, 120))
    else:
        title = title_font.render("游戏结束", True, LOSS_COLOR)
        subtitle = main_font.render("你未能实现利润目标", True, TEXT_COLOR)
        achievement = main_font.render("请总结经验，重新开始！", True, (220, 200, 120))
    
    surface.blit(title, (WIDTH//2 - title.get_width()//2, HEIGHT//2 - 100))
    surface.blit(subtitle, (WIDTH//2 - subtitle.get_width()//2, HEIGHT//2 - 40))
    surface.blit(achievement, (WIDTH//2 - achievement.get_width()//2, HEIGHT//2 + 10))
    
    # 最终统计
    stats = [
        f"总周数: {game.week}",
        f"最高单周利润: ${max(game.profit_history) if game.profit_history else 0:,.2f}",
        f"平均利润率: {sum(game.profit_margin_history)/len(game.profit_margin_history):.1f}%",
        f"连续达标周数: {game.consecutive_weeks}"
    ]
    
    for i, stat in enumerate(stats):
        stat_text = main_font.render(stat, True, TEXT_COLOR)
        surface.blit(stat_text, (WIDTH//2 - stat_text.get_width()//2, HEIGHT//2 + 50 + i*30))

# 主游戏函数
def main():
    game = ElasticMarketGame()
    clock = pygame.time.Clock()
    
    # 创建按钮
    next_week_btn = Button(WIDTH - 200, HEIGHT - 70, 180, 50, "进入下一周")
    restart_btn = Button(WIDTH//2 - 90, HEIGHT//2 + 120, 180, 50, "重新开始")
    
    # 创建滑块
    slider = Slider(0, 0, 300, 30)
    
    # 初始计算利润
    game.calculate_total_profit()
    
    # 应用调整按钮
    apply_btn = None
    
    running = True
    while running:
        mouse_pos = pygame.mouse.get_pos()
        
        for event in pygame.event.get():
            if event.type == QUIT:
                running = False
                
            # 处理按钮事件
            next_week_btn.handle_event(event)
            if game.game_state != "playing":
                restart_btn.handle_event(event)
            
            # 处理应用调整按钮
            if apply_btn:
                apply_btn.handle_event(event)
                
            # 处理滑块事件
            slider.handle_event(event)
            
            if event.type == MOUSEBUTTONDOWN and event.button == 1:
                # 检查是否点击了商品选择按钮
                col_widths = [
                    int(900 * 0.15),  # 商品
                    int(900 * 0.10),  # 类别
                    int(900 * 0.12),  # 当前价格
                    int(900 * 0.12),  # 当前销量
                    int(900 * 0.18),  # 建议价格区间
                    int(900 * 0.13),  # 利润率
                    int(900 * 0.15)   # 操作
                ]
                
                start_y = 150
                for idx in range(len(game.products)):
                    row_y = start_y + idx * 40
                    cum_col = sum(col_widths[0:6])  # 操作按钮在第6列
                    button_rect = pygame.Rect(50 + cum_col + 10, row_y + 5, 100, 30)
                    if button_rect.collidepoint(mouse_pos):
                        game.selected_product = idx
                        slider.value = 0  # 重置滑块
                
            if event.type == KEYDOWN:
                if event.key == K_RETURN and game.adjustments_left > 0:
                    game.adjust_price(game.selected_product, slider.value)
                    slider.value = 0
                elif event.key == K_SPACE:
                    game.next_week()
                elif event.key == K_r:
                    game = ElasticMarketGame()
                    slider.value = 0
                    game.calculate_total_profit()
        
        # 处理按钮悬停
        next_week_btn.check_hover(mouse_pos)
        restart_btn.check_hover(mouse_pos)
        
        # 处理"进入下一周"按钮动作
        if next_week_btn.action is None:
            next_week_btn.action = game.next_week
        
        # 处理"重新开始"按钮动作
        if restart_btn.action is None:
            restart_btn.action = lambda: globals().update(game=ElasticMarketGame(), slider=Slider(0,0,300,30))
        
        # 绘制界面
        screen.fill(BACKGROUND)
        
        # 绘制标题
        title = title_font.render("弹性市场大亨 - 经济学模拟游戏", True, HIGHLIGHT)
        subtitle = small_font.render("通过调整商品价格应对市场竞争，理解价格需求弹性差异", True, TEXT_COLOR)
        screen.blit(title, (WIDTH//2 - title.get_width()//2, 15))
        screen.blit(subtitle, (WIDTH//2 - subtitle.get_width()//2, 60))
        
        # 绘制仪表盘
        dashboard_bottom = draw_dashboard(screen, game, 50, 100, 900, 400)
        
        # 绘制公式面板
        draw_elasticity_formula(screen, 50, dashboard_bottom + 20)
        
        # 绘制事件面板
        draw_event_panel(screen, game, 470, dashboard_bottom + 20, 480, 160)
        
        # 绘制游戏状态
        draw_game_status(screen, game, 50, dashboard_bottom + 180, 400, 260)
        
        # 绘制价格调整面板并获取应用按钮
        apply_btn = draw_price_adjustment(screen, game, slider, 470, dashboard_bottom + 190, 480, 250)
        
        # 绘制"进入下一周"按钮
        next_week_btn.draw(screen)
        
        # 处理应用按钮悬停
        if apply_btn:
            apply_btn.check_hover(mouse_pos)
        
        # 绘制游戏结束画面
        if game.game_state != "playing":
            draw_game_over(screen, game)
            restart_btn.draw(screen)
        
        pygame.display.flip()
        clock.tick(60)
    
    pygame.quit()
    sys.exit()

if __name__ == "__main__":
    main()
