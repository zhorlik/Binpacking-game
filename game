#We used pygame to create a 2D bin packing game. We have a menu screen where the user can choose to play the game, go to settings or quit the game.
#In the game, the user has to drag and drop items into bins. The user can add or remove bins, reset the items, check if the items are placed
#correctly, and go back to the menu. We implemented the first fit decreasing and best fit decreasing algorithms so the user can also see
#the optimal solution for the current items by using the FFD or BFD buttons (to see the difference between these two algorithms, we recommend
#using a large number of large items with smaller bins).

import pygame
import random

#Define colors
WHITE = (255, 255, 255)
LIGHT_GRAY = (200, 200, 200)
GRAY = (150, 150, 150)
BLACK = (0, 0, 0)

#Define initial item and bin characteristics
global_num_items = 30
global_item_size = 50
global_bin_size_x = 100
global_bin_size_y = 200

start_time = 0

#Button class
class Button:
    def __init__(self, screen, x, y, width, height, text, text_color, button_color, hover_color):
        self.screen = screen
        self.rect = pygame.Rect(x, y, width, height)
        self.text = text
        self.text_color = text_color
        self.button_color = button_color
        self.hover_color = hover_color
        self.font = pygame.font.Font(None, 20)
        self.hovered = False #Will be True if the mouse is over the button

    def draw(self): #Draw the button with hover effect
        if self.hovered:
            pygame.draw.rect(self.screen, self.hover_color, self.rect)
        else:
            pygame.draw.rect(self.screen, self.button_color, self.rect)
        text_surface = self.font.render(self.text, True, self.text_color)
        text_rect = text_surface.get_rect(center=self.rect.center)
        self.screen.blit(text_surface, text_rect)

    def is_clicked(self, mouse_pos): #Check if the button is clicked
        return self.rect.collidepoint(mouse_pos)

    def update(self, mouse_pos): #Update button based on mouse position
        self.hovered = self.rect.collidepoint(mouse_pos)

#Define a Bin class to manage storage units where the items will be placed
class Bin:
    def __init__(self, screen, x, y, width, height):
        self.screen = screen
        self.width = width
        self.height = height
        self.rect = pygame.Rect(x, y, width, height)
        self.border = pygame.Rect(x - 5, y - 5, width + 10, height + 10)  #Border around the bin to prevent items from going outside

    def draw(self): #Draw the bin on the screen
        pygame.draw.rect(self.screen, LIGHT_GRAY, self.rect)

    def draw_grid(self, grid_size): #Draw grid inside the bin to help with item placement
        for x in range(self.rect.x, self.rect.x + self.width, grid_size):
            pygame.draw.line(self.screen, GRAY, (x, self.rect.y), (x, self.rect.y + self.rect.height))
        for y in range(self.rect.y, self.rect.y + self.rect.height, grid_size):
            pygame.draw.line(self.screen, GRAY, (self.rect.x, y), (self.rect.x + self.width, y))

    def get_grid_position(self, item): #Get grid position for placing items neatly in the bin
            grid_x = round((item.rect.x - self.rect.x) / 10) * 10
            grid_y = round((item.rect.y - self.rect.y) / 10) * 10
            return grid_x + self.rect.x, grid_y + self.rect.y

#Starting area to display the items at the beggining of the game
class Starea:
    def __init__(self, screen, width, height):
        self.screen = screen
        self.width = width
        self.height = height
        self.rect = pygame.Rect(0, 500, width, height)
        self.border = pygame.Rect(0 - 5, 500 - 5, width + 10, height + 10)

    def draw(self): #Draw the starting area on the screen (it will be transparent)
        pygame.draw.rect(self.screen, WHITE, self.rect)

    def get_grid_position(self, item): #Get grid position for placing items neatly in the starting area
            grid_x = round((item.rect.x - self.rect.x) / 10) * 10
            grid_y = round((item.rect.y - self.rect.y) / 10) * 10
            return grid_x + self.rect.x, grid_y + self.rect.y

#Item class
class Item:
    def __init__(self, screen, x, y, width, height, color):
        self.screen = screen
        self.width = width
        self.height = height
        self.color = color
        self.rect = pygame.Rect(x, y, width, height)
        self.dragging = False
        self.offset_x = 0
        self.offset_y = 0

    def draw(self): #Draw the item on the screen
        pygame.draw.rect(self.screen, self.color, self.rect)

    def start_drag(self, mouse_x, mouse_y, bins): #Start dragging the item
        if self.rect.collidepoint(mouse_x, mouse_y):
            self.dragging = True
            self.offset_x = mouse_x - self.rect.x
            self.offset_y - mouse_y - self.rect.y

    def drag(self, mouse_x, mouse_y, bins): #Handle the dragging of the item
        if self.dragging:
            self.rect.x = mouse_x - self.offset_x
            self.rect.y = mouse_y - self.offset_y
            for bin in bins:
                if self.rect.colliderect(bin.border):  #Check collision with bin's border
                    if self.rect.left < bin.rect.left:
                        self.rect.left = bin.rect.left
                    elif self.rect.right > bin.rect.right:
                        self.rect.right = bin.rect.right
                    if self.rect.top < bin.rect.top:
                        self.rect.top = bin.rect.top
                    elif self.rect.bottom > bin.rect.bottom:
                        self.rect.bottom = bin.rect.bottom

    def stop_drag(self, bins, starea): #Stop dragging the item and snap it to the grid
        self.dragging = False
        for bin in bins:
            if self.rect.colliderect(bin.rect):
                grid_x, grid_y = bin.get_grid_position(self)
                self.rect.x = grid_x
                self.rect.y = grid_y
        if self.rect.colliderect(starea.rect):
            grid_x, grid_y = starea.get_grid_position(self)
            self.rect.x = grid_x
            self.rect.y = grid_y

#First Fit Decreasing algorithm
def first_fit_decreasing(items, bins, screen, bin_size_x, bin_size_y):
    opt_bins_used = 0
    items_sorted = sorted(items, key=lambda i: i.height * i.width, reverse=True) #Sort items by decreasing area
    for item in items_sorted:
        placed = False
        while not placed:
            for bin in bins: #Try to place the item in an existing bin following the FFD rules
                if not placed:
                    for x in range(bin.rect.x, bin.rect.x + bin.width - item.width + 1, 10):
                        for y in range(bin.rect.y, bin.rect.y + bin.height - item.height + 1, 10):
                            temp_rect = pygame.Rect(x, y, item.width, item.height)
                            if all(not temp_rect.colliderect(other.rect) for other in items):
                                item.rect.x = x
                                item.rect.y = y
                                placed = True
                                break
                        if placed:
                            break
            if not placed: #Create a new bin if no existing bin can accommodate the item
                new_x = 100 + len(bins) * (bin_size_x + 50)
                bins.append(Bin(screen, new_x, 200, bin_size_x, bin_size_y))
                opt_bins_used += 1


    return opt_bins_used

#This is an algorithm that that will be used to fit the items in the starting area at the beginning of the game
def fitting_in_starea(items, starea, screen):
    items_sorted = sorted(items, key=lambda i: i.height * i.width, reverse=True) #Sort items by decreasing area
    for item in items_sorted:
        placed = False
        while not placed:
            for x in range(starea.rect.x, starea.rect.x + starea.width - item.width + 1, 10):
                for y in range(starea.rect.y, starea.rect.y + starea.height - item.height + 1, 10):
                    temp_rect = pygame.Rect(x, y, item.width, item.height)
                    if all(not temp_rect.colliderect(other.rect) for other in items):
                        item.rect.x = x
                        item.rect.y = y
                        placed = True
                        break
                if placed:
                    break
            if not placed: #really similar to FFD until this point; instead of adding a bin we will increase the size of the starting area
                starea = Starea(screen, starea.width + 100, starea.height)

#Best Fit Decreasing algorithm
def best_fit_decreasing(items, bins, screen, bin_size_x, bin_size_y):
    items_sorted = sorted(items, key=lambda i: i.height * i.width, reverse=True) #Sort items by decreasing area
    remaining_space_dict = {} #Dictionary to keep track of the remaining space in each bin

    for item in items_sorted:
        best_bin = None
        best_fit_space = float('inf')

        for i, bin in enumerate(bins): #Try to find the best bin to place the item in
            if remaining_space_dict.get(i) is not None:
                for x in range(bin.rect.x, bin.rect.x + bin.width - item.width + 1, 10):
                    for y in range(bin.rect.y, bin.rect.y + bin.height - item.height + 1, 10):
                        temp_rect = pygame.Rect(x, y, item.width, item.height)
                        if all(not temp_rect.colliderect(other.rect) for other in items if other != item):
                            remaining_space = remaining_space_dict[i] - item.width * item.height #Calculate the remaining space in the bin
                            if remaining_space < best_fit_space:
                                best_fit_space = remaining_space
                                best_bin = bin
                                best_position = (x, y)

        if best_bin: #Place the item in the best bin
            item.rect.x, item.rect.y = best_position
            remaining_space_dict[bins.index(best_bin)] = best_fit_space
        else: #Create a new bin if no existing bin can accommodate the item
            new_x = 100 + len(bins) * (bin_size_x + 50)
            new_bin = Bin(screen, new_x, 200, bin_size_x, bin_size_y)
            bins.append(new_bin)
            remaining_space_dict[len(bins) - 1] = bin_size_x * bin_size_y - item.width * item.height
            item.rect.x, item.rect.y = (new_bin.rect.x, new_bin.rect.y)

pygame.init()

#Set up the screen
screen_width = 1200
screen_height = 600
screen = pygame.display.set_mode((screen_width, screen_height))
pygame.display.set_caption("Menu")
#Clock for controlling the frame rate
clock = pygame.time.Clock()

#Play screen for actually playing the game
def play(num_items, item_size, bin_size_x, bin_size_y):

    #Create buttons to add and remove bins
    add_bin_button = Button(screen, 100, 50, 100, 50, "Add Bin", (255, 255, 255), (0, 0, 255), (0, 0, 128))
    remove_bin_button = Button(screen, 225, 50, 100, 50, "Remove Bin", (255, 255, 255), (255, 0, 0), (128, 0, 0))
    reset_items_button = Button(screen, 350, 50, 100, 50, "Reset Items", (255, 255, 255), (0, 0, 0), (128, 128, 128))
    ffd_show_optimal_button = Button(screen, 475, 50, 100, 50, "FFD Solution", (255, 255, 255), (128, 0, 128), (64, 0, 64))
    bfd_show_optimal_button = Button(screen, 600, 50, 100, 50, "BFD Solution", (255, 255, 255), (128, 128, 128), (64, 64, 64))
    check_button = Button(screen, 725, 50, 100, 50, "Check", (255, 255, 255), (0, 255, 0), (0, 128, 0))
    back_to_menu_button = Button(screen, 850, 50, 100, 50, "Back to Menu", (255, 255, 255), (255, 165, 0), (165, 42, 42))

    #Create initial bins and items
    bins = []
    items = []
    possible_item_size = [] #Items will have random sizes in the given range
    for i in range(10, item_size, 10):
        possible_item_size.append(i)

    for _ in range(num_items): #Create the items
        x = random.randint(50, 150)
        y = random.randint(50, 150)
        width = random.choice(possible_item_size)
        height = random.choice(possible_item_size)
        color = (random.randint(0, 255), random.randint(0, 255), random.randint(0, 255))
        items.append(Item(screen, x, y, width, height, color))

    opt_bins_used = first_fit_decreasing(items, bins, screen, bin_size_x, bin_size_y) #calling an initial FFD to get the optimal number of bins
    print(f"Optimal number of bins: {opt_bins_used}")

    bins.clear() #reset bins post calculation
    for item in items: #reset items post calculation
        item.rect.x = random.randint(50, 150)
        item.rect.y = random.randint(50, 150)

    starea = Starea(screen, 100, 100) #Create the starting area
    fitting_in_starea(items, starea, screen) #Fit the items in the starting area

    #Main loop
    running = True
    while running:
        screen.fill(WHITE)
        mouse_pos = pygame.mouse.get_pos()
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False
            elif event.type == pygame.MOUSEBUTTONDOWN and event.button == 1:

                if add_bin_button.is_clicked(event.pos): #Add a new bin
                    new_x = 100 + len(bins) * 150
                    bins.append(Bin(screen, new_x, 200, bin_size_x, bin_size_y))

                elif remove_bin_button.is_clicked(event.pos) and bins: #Remove the last bin
                    bins.pop()

                elif reset_items_button.is_clicked(event.pos): #Reset the items in the starting area
                    for item in items:
                        item.rect.x = random.randint(50, 150)
                        item.rect.y = random.randint(50, 150)
                    fitting_in_starea(items, starea, screen)

                elif ffd_show_optimal_button.is_clicked(event.pos): #Show the optimal solution using FFD
                    for item in items:
                        item.rect.x = random.randint(50, 150)
                        item.rect.y = random.randint(50, 150)
                    fitting_in_starea(items, starea, screen)
                    bins.clear()
                    first_fit_decreasing(items, bins, screen, bin_size_x, bin_size_y)

                elif bfd_show_optimal_button.is_clicked(event.pos): #Show the optimal solution using BFD
                    for item in items:
                        item.rect.x = random.randint(50, 150)
                        item.rect.y = random.randint(50, 150)
                    fitting_in_starea(items, starea, screen)
                    bins.clear()
                    best_fit_decreasing(items, bins, screen, bin_size_x, bin_size_y)

                elif check_button.is_clicked(event.pos): #Check if the user has placed the items correctly
                    #Check if all items are placed in the bins and if they overlap
                    all_items_placed = all(any(item.rect.colliderect(bin.rect) for bin in bins) for item in items)
                    items_overlap = any(item1.rect.colliderect(item2.rect) for item1 in items for item2 in items if item1 != item2)
                    if all_items_placed and not items_overlap:
                        if len(bins) <= opt_bins_used: #Check if the user has used the optimal number of bins
                            winner_screen() #Show the winner screen if the user has placed the items correctly and used the optimal number of bins
                        else:
                            too_many_bins_screen() #Show the too many bins screen if the user has used too many bins
                    elif items_overlap:
                        items_overlap_screen() #Show the items overlap screen if the user has placed items on top of each other
                    else:
                        not_all_items_are_placed_screen() #Show the not all items are placed screen if the user has not placed all items

                elif back_to_menu_button.is_clicked(event.pos): #Go back to the menu
                    main_menu()

                for item in items: #start dragging upon mouse click
                    item.start_drag(event.pos[0], event.pos[1], bins)
            elif event.type == pygame.MOUSEMOTION:  #handle dragging the items
                for item in items:
                    item.drag(event.pos[0], event.pos[1], bins)
            elif event.type == pygame.MOUSEBUTTONUP and event.button == 1: #stop dragging upon mouse release
                for item in items:
                    item.stop_drag(bins, starea)

        #Clear the screen
        screen.fill(WHITE)

        #Draw bins and grids
        for bin in bins:
            bin.draw()
            bin.draw_grid(10)  # Draw grid inside bins

        #Draw items
        for item in items:
            item.draw()

        add_bin_button.update(mouse_pos)
        remove_bin_button.update(mouse_pos)
        reset_items_button.update(mouse_pos)
        ffd_show_optimal_button.update(mouse_pos)
        bfd_show_optimal_button.update(mouse_pos)
        check_button.update(mouse_pos)
        back_to_menu_button.update(mouse_pos)
        # Draw the buttons
        add_bin_button.draw()
        remove_bin_button.draw()
        reset_items_button.draw()
        ffd_show_optimal_button.draw()
        bfd_show_optimal_button.draw()
        check_button.draw()
        back_to_menu_button.draw()

        pygame.display.update()

    pygame.quit()

#Main menu function
def main_menu():
    #Create buttons
    play_button = Button(screen, 200, 225, 200, 150, "Play", (255, 255, 255), (0, 0, 255), (0, 0, 128))
    settings_button = Button(screen, 500, 225, 200, 150, "Settings", (255, 255, 255), (0, 0, 0), (128, 128, 128))
    quit_button = Button(screen, 800, 225, 200, 150, "Quit", (255, 255, 255), (255, 0, 0), (128, 0, 0))


    running = True
    while running:
        screen.fill(WHITE)
        mouse_pos = pygame.mouse.get_pos()
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False
            elif event.type == pygame.MOUSEBUTTONDOWN and event.button == 1:
                if play_button.is_clicked(event.pos): #Start the game
                    play(global_num_items, global_item_size, global_bin_size_x, global_bin_size_y)
                elif settings_button.is_clicked(event.pos): #Go to the settings
                    settings()
                elif quit_button.is_clicked(event.pos): #Quit the game
                    running = False

        play_button.update(mouse_pos)
        settings_button.update(mouse_pos)
        quit_button.update(mouse_pos)
        #Draw the buttons
        play_button.draw()
        settings_button.draw()
        quit_button.draw()


        pygame.display.update()

    pygame.quit()

#Settings fuction to configure main options
def settings():
    global global_num_items
    global global_item_size
    global global_bin_size_x
    global global_bin_size_y

    #Create buttons
    num_items_label = Button(screen, 10, 50, 100, 50, "Number of items", (0, 0, 0), (255, 255, 255), (255, 255, 255))
    item_size_label = Button(screen, 10, 150, 100, 50, "Item size", (0, 0, 0), (255, 255, 255), (255, 255, 255))
    bin_size_label = Button(screen, 10, 250, 100, 50, "Bin size", (0, 0, 0), (255, 255, 255), (255, 255, 255))
    back_button = Button(screen, 200, 500, 100, 50, "Back", (255, 255, 255), (0, 0, 0), (128, 128, 128))

    #Current setting indicators
    current_num_items = Button(screen, 1000, 50, 100, 50, f"Current: {global_num_items}", (0, 0, 0), (255, 255, 255), (255, 255, 255))
    current_item_size = Button(screen, 1000, 150, 100, 50, f"Current: {global_item_size}", (0, 0, 0), (255, 255, 255), (255, 255, 255))
    current_bin_size = Button(screen, 1000, 250, 100, 50, f"Current: {global_bin_size_x}x{global_bin_size_y}", (0, 0, 0), (255, 255, 255), (255, 255, 255))

    #Buttons for changing settings
    num_items_buttons = [Button(screen, 200 + i * 125, 50, 100, 50, str(10 + 10 * i), (255, 255, 255), (0, 0, 0), (128, 128, 128)) for i in range(5)]
    item_size_buttons = [Button(screen, 200 + i * 125, 150, 100, 50, str(20 + 10 * i), (255, 255, 255), (0, 0, 0), (128, 128, 128)) for i in range(5)]
    bin_size_buttons = [
        Button(screen, 200, 250, 150, 50, "50x100", (255, 255, 255), (0, 0, 0), (128, 128, 128)),
        Button(screen, 375, 250, 150, 50, "100x100", (255, 255, 255), (0, 0, 0), (128, 128, 128)),
        Button(screen, 550, 250, 150, 50, "100x200", (255, 255, 255), (0, 0, 0), (128, 128, 128))
    ]

    running = True
    while running:
        screen.fill(WHITE)
        mouse_pos = pygame.mouse.get_pos()

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False
            elif event.type == pygame.MOUSEBUTTONDOWN and event.button == 1:
                if back_button.is_clicked(event.pos):
                    main_menu()
                    break

                #Update settings upon mouse click
                for button in num_items_buttons:
                    if button.is_clicked(event.pos):
                        global_num_items = int(button.text)
                        current_num_items.text = f"Current: {global_num_items}"

                for button in item_size_buttons:
                    if button.is_clicked(event.pos):
                        global_item_size = int(button.text)
                        current_item_size.text = f"Current: {global_item_size}"

                for button in bin_size_buttons:
                    if button.is_clicked(event.pos):
                        sizes = button.text.split('x')
                        global_bin_size_x, global_bin_size_y = int(sizes[0]), int(sizes[1])
                        current_bin_size.text = f"Current: {global_bin_size_x}x{global_bin_size_y}"

        #Update and draw all buttons
        back_button.update(mouse_pos)
        back_button.draw()

        for button in num_items_buttons + item_size_buttons + bin_size_buttons:
            button.update(mouse_pos)
            button.draw()

        num_items_label.draw()
        item_size_label.draw()
        bin_size_label.draw()
        current_num_items.draw()
        current_item_size.draw()
        current_bin_size.draw()


        pygame.display.update()

    pygame.quit()

#Screens for different game outcomes
def winner_screen():
    global start_time
    start_time = pygame.time.get_ticks()
    running = True
    while running:
        screen.fill(WHITE)
        font = pygame.font.Font(None, 36)
        text = font.render("Congratulations! You won!", True, (0, 0, 0))
        text_rect = text.get_rect(center=(screen_width // 2, screen_height // 2))
        screen.blit(text, text_rect)
        pygame.display.update()
        current_time = pygame.time.get_ticks()
        if current_time - start_time >= 4000:
            return  #Return to the play screen after 5 seconds
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                quit()

def too_many_bins_screen():
    global start_time
    start_time = pygame.time.get_ticks()
    running = True
    while running:
        screen.fill(WHITE)
        font = pygame.font.Font(None, 36)
        text = font.render("You used too many bins... get back to work!", True, (0, 0, 0))
        text_rect = text.get_rect(center=(screen_width // 2, screen_height // 2))
        screen.blit(text, text_rect)
        pygame.display.update()
        current_time = pygame.time.get_ticks()
        if current_time - start_time >= 4000:
            return  #Return to the play screen after 5 seconds
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                quit()

def items_overlap_screen():
    global start_time
    start_time = pygame.time.get_ticks()
    running = True
    while running:
        screen.fill(WHITE)
        font = pygame.font.Font(None, 36)
        text = font.render("Stacking items on top of each other? This isn't Jenga, my friend.", True, (0, 0, 0))
        text_rect = text.get_rect(center=(screen_width // 2, screen_height // 2))
        screen.blit(text, text_rect)
        pygame.display.update()
        current_time = pygame.time.get_ticks()
        if current_time - start_time >= 4000:
            return  #Return to the play screen after 5 seconds
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                quit()

def not_all_items_are_placed_screen():
    global start_time
    start_time = pygame.time.get_ticks()
    running = True
    while running:
        screen.fill(WHITE)
        font = pygame.font.Font(None, 36)
        text = font.render("Leaving items untouched, are we? Keeping things minimalist, huh?", True, (0, 0, 0))
        text_rect = text.get_rect(center=(screen_width // 2, screen_height // 2))
        screen.blit(text, text_rect)
        pygame.display.update()
        current_time = pygame.time.get_ticks()
        if current_time - start_time >= 5000:
            return  #Return to the play screen after 5 seconds
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                quit()

main_menu() #Start the game by displaying the main menu
