# Game Development Guide

Welcome to this comprehensive guide on developing various components of a game. This document will walk you through the fundamental concepts and provide code examples for essential game systems.

## Table of Contents

1.  [Sprite Renderer](#1-sprite-renderer)
2.  [Physics](#2-physics)
3.  [Animation](#3-animation)
4.  [Tilemap Renderer](#4-tilemap-renderer)
5.  [Collision Handling](#5-collision-handling)
6.  [Audio Player](#6-audio-player)
7.  [YAML Serialization](#7-yaml-serialization)

---

## 1. Sprite Renderer

### Conceptual Overview

A Sprite Renderer is responsible for drawing 2D images (sprites) onto the screen. Sprites are the visual building blocks for most 2D games, representing characters, objects, UI elements, etc. The renderer needs to handle aspects like position, rotation, scale, and transparency of sprites.

### How to Code and Develop

Developing a sprite renderer typically involves the following steps:

1.  **Load Textures:** Sprites are usually derived from texture files (e.g., PNG, JPG). You'll need a mechanism to load these textures into memory.
2.  **Define Sprite Data:** Create a data structure or class to hold information about each sprite, such as its texture, position, rotation, scale, color tint, and drawing order (layer).
3.  **Rendering Loop:** In your game's main loop, iterate through all active sprites and draw them. This often involves using a graphics library (like OpenGL, DirectX, SDL, SFML, or a game engine's built-in renderer).
4.  **Transformations:** Apply transformations (translation, rotation, scaling) to sprites before drawing them. This is usually done using matrices.
5.  **Batching (Optional but Recommended):** To improve performance, group sprites that use the same texture or shader and draw them in a single batch call to the graphics API.

### Code Example (Conceptual - using a hypothetical Python-like graphics library)

```python
class Texture:
    def __init__(self, filepath):
        # In a real scenario, this would load image data
        self.image_data = self._load_image(filepath)
        self.width, self.height = self._get_dimensions(self.image_data)
        print(f"Texture '{filepath}' loaded ({self.width}x{self.height})")

    def _load_image(self, filepath):
        # Placeholder for actual image loading logic
        # e.g., using libraries like Pillow (PIL) in Python
        # or platform-specific APIs
        return f"Image data for {filepath}"

    def _get_dimensions(self, image_data):
        # Placeholder for getting dimensions
        return 100, 100 # Example dimensions

class Sprite:
    def __init__(self, texture, position=(0, 0), scale=(1, 1), rotation=0.0, tint=(255, 255, 255, 255)):
        self.texture = texture
        self.position = list(position) # (x, y)
        self.scale = list(scale)       # (sx, sy)
        self.rotation = rotation     # degrees
        self.tint = list(tint)         # (r, g, b, a)
        self.visible = True
        self.source_rect = None # Added for sprite sheet animation (x, y, width, height)

class SpriteRenderer:
    def __init__(self, graphics_api):
        self.graphics_api = graphics_api # Hypothetical graphics API object
        self.sprites_to_render = []

    def add_sprite(self, sprite):
        self.sprites_to_render.append(sprite)

    def remove_sprite(self, sprite):
        if sprite in self.sprites_to_render:
            self.sprites_to_render.remove(sprite)

    def render(self):
        # Sort sprites by layer or depth if necessary (not shown here)
        for sprite in self.sprites_to_render:
            if not sprite.visible:
                continue

            # In a real renderer, you'd set up transformation matrices
            # and then make a draw call.
            # If sprite.source_rect is defined, draw only that part of the texture.
            if sprite.source_rect:
                self.graphics_api.draw_texture_part(
                    sprite.texture,
                    sprite.position,
                    sprite.rotation,
                    sprite.scale,
                    sprite.tint,
                    sprite.source_rect # (sx, sy, sw, sh) from the texture
                )
            else:
                self.graphics_api.draw_texture_full(
                    sprite.texture,
                    sprite.position,
                    sprite.rotation,
                    sprite.scale,
                    sprite.tint
                )
        # If using batching, this is where you'd flush the batch
        # self.graphics_api.flush_batch()

# Example Usage (Conceptual)
# class HypotheticalGraphicsAPI:
#     def draw_texture_full(self, texture, position, rotation, scale, tint):
#         print(f"Drawing FULL '{texture.image_data}' at {position} rot {rotation} scale {scale} tint {tint}")
#     def draw_texture_part(self, texture, position, rotation, scale, tint, source_rect):
#         print(f"Drawing PART of '{texture.image_data}' (rect: {source_rect}) at {position} rot {rotation} scale {scale} tint {tint}")


# graphics = HypotheticalGraphicsAPI()
# renderer = SpriteRenderer(graphics)

# player_texture = Texture("player.png")
# player_sprite = Sprite(player_texture, position=(100, 150))
# renderer.add_sprite(player_sprite)

# In game loop:
# renderer.render()
```

**Explanation:**

*   `Texture`: Represents an image loaded from a file.
*   `Sprite`: Holds all the properties of a single sprite instance, including an optional `source_rect` for drawing portions of a texture (used by animation).
*   `SpriteRenderer`: Manages a list of sprites and uses a `graphics_api` to draw them. It checks for `source_rect` to decide whether to draw the full texture or a part of it.
*   The `render` method iterates through visible sprites and calls the graphics API to draw each one. Real-world implementations would involve complex matrix math for transformations and potentially sophisticated batching systems.

---

## 2. Physics

### Conceptual Overview

A game physics system simulates realistic motion and interactions between game objects. This can range from simple movement and gravity to complex rigid body dynamics, soft body simulations, and fluid dynamics. For most 2D games, a 2D rigid body physics engine is common.

Key concepts:

*   **Rigid Body:** An object that does not deform. It has properties like mass, velocity, acceleration, friction, and restitution (bounciness).
*   **Forces:** Influences that can change a body's motion (e.g., gravity, thrust, springs).
*   **Impulses:** Instantaneous changes in velocity (e.g., from a collision).
*   **Collision Detection:** Identifying when and where objects intersect (covered more in "Collision Handling").
*   **Collision Response:** Determining how objects react after a collision (e.g., bouncing off, stopping).
*   **Integration:** The numerical method used to update an object's position and orientation over time based on its velocity and acceleration (e.g., Euler integration, Verlet integration).

### How to Code and Develop

1.  **Define Rigid Body Component:** Create a class or structure to represent a rigid body with its physical properties.
2.  **Physics World/Engine:** This class will manage all rigid bodies, apply global forces (like gravity), and step the simulation forward in time.
3.  **Integrator:** Implement an integration method. Euler integration is simple to start with:
    *   `velocity += acceleration * deltaTime`
    *   `position += velocity * deltaTime`
4.  **Force Accumulation:** Before each integration step, sum all forces acting on each body.
5.  **Collision Detection and Response:** Integrate with your collision handling system. When collisions occur, apply appropriate responses (e.g., impulse-based resolution).

### Code Example (Simple 2D Rigid Body and Euler Integration)

```python
import math

class Vec2:
    def __init__(self, x=0.0, y=0.0):
        self.x = float(x)
        self.y = float(y)

    def __add__(self, other):
        return Vec2(self.x + other.x, self.y + other.y)

    def __sub__(self, other):
        return Vec2(self.x - other.x, self.y - other.y)

    def __mul__(self, scalar):
        return Vec2(self.x * float(scalar), self.y * float(scalar))

    def __truediv__(self, scalar):
        if scalar == 0: return Vec2(0,0) # Avoid division by zero
        return Vec2(self.x / float(scalar), self.y / float(scalar))

    def length(self):
        return math.sqrt(self.x**2 + self.y**2)

    def normalize(self):
        l = self.length()
        if l == 0: return Vec2(0,0)
        return Vec2(self.x / l, self.y / l)

    def __str__(self):
        return f"Vec2({self.x:.2f}, {self.y:.2f})"

class RigidBody2D:
    def __init__(self, mass=1.0, position=None, velocity=None):
        self.mass = float(mass)
        self.inverse_mass = 1.0 / self.mass if self.mass != 0 else 0.0 # Infinite mass for static objects
        self.position = position if position is not None else Vec2(0,0)
        self.velocity = velocity if velocity is not None else Vec2(0,0)
        self.acceleration = Vec2(0,0) # Current acceleration
        self.force_accumulator = Vec2(0,0) # Sum of forces for this frame
        self.friction_coefficient = 0.1 # Simple air friction coefficient
        self.restitution = 0.5 # Bounciness (0 to 1)

    def add_force(self, force_vector):
        self.force_accumulator += force_vector

    def clear_forces(self):
        self.force_accumulator = Vec2(0,0)

    def integrate(self, dt): # dt is delta time (time since last frame)
        if self.inverse_mass == 0: # Static body
            self.velocity = Vec2(0,0) # Static bodies don't move
            return

        # Apply accumulated forces to get acceleration (F=ma -> a=F/m)
        self.acceleration = self.force_accumulator * self.inverse_mass

        # Simple Euler integration
        self.velocity += self.acceleration * dt
        self.position += self.velocity * dt

        # Apply some damping/friction (optional, can be more sophisticated)
        # This is a simple air drag model
        if self.velocity.length() > 0: # only apply if moving
             friction_force = self.velocity.normalize() * -1 * self.friction_coefficient * self.velocity.length()**2 # F_drag = -k * v^2
             self.add_force(friction_force) # This force will be applied in the next integration step's acceleration calculation if not cleared immediately

        # Clear forces for the next frame AFTER they've been used for acceleration
        self.clear_forces()


class PhysicsWorld:
    def __init__(self, gravity=None): # Typical gravity, y-down
        self.bodies = []
        self.gravity = gravity if gravity is not None else Vec2(0, 9.81)


    def add_body(self, body):
        self.bodies.append(body)

    def remove_body(self, body):
        if body in self.bodies:
            self.bodies.remove(body)

    def step(self, dt):
        for body in self.bodies:
            if body.inverse_mass == 0: # Don't apply gravity to static/kinematic objects with infinite mass
                body.clear_forces() # Ensure static bodies don't accumulate forces if any were added
                continue
            # Apply gravity (mass dependent)
            body.add_force(self.gravity * body.mass)

        # Integration step
        for body in self.bodies:
            body.integrate(dt) # Forces are cleared inside integrate after use

        # Collision detection and response would go here
        # self.handle_collisions(dt)

# Example Usage:
# world = PhysicsWorld(gravity=Vec2(0, 981)) # Pixels/second^2
# ball = RigidBody2D(mass=1.0, position=Vec2(100, 50))
# world.add_body(ball)
# ground = RigidBody2D(mass=0, position=Vec2(0, 500)) # Static body
# world.add_body(ground)


# In game loop:
# dt = 1.0 / 60.0 # Assuming 60 FPS
# world.step(dt)
# print(f"Ball position: {ball.position}, velocity: {ball.velocity}")
```

**Explanation:**

*   `Vec2`: A simple 2D vector class for positions, velocities, forces, with added methods like `length`, `normalize`, and `__truediv__`.
*   `RigidBody2D`: Stores physical properties. `add_force` accumulates forces, and `integrate` updates position and velocity using Euler integration. Friction is conceptualized and would typically be part of force accumulation or a direct velocity modification.
*   `PhysicsWorld`: Manages `RigidBody2D` objects, applies global forces like gravity, and calls `integrate` on each body.
*   The `integrate` method uses Euler integration. More advanced integrators like Verlet or Runge-Kutta provide better stability and accuracy, especially with larger time steps or more complex forces.

---

## 3. Animation

### Conceptual Overview

Sprite animation creates the illusion of movement by rapidly displaying a sequence of slightly different images (frames). This is commonly used for character movements, special effects, and dynamic UI elements.

Key concepts:

*   **Frames:** Individual images in an animation sequence.
*   **Sprite Sheet:** A single image file containing multiple frames of an animation, often arranged in a grid. This is efficient for texture memory and rendering.
*   **Animation Clip:** A defined sequence of frames from a sprite sheet, along with timing information (e.g., duration per frame or total duration).
*   **Animator/Animation Controller:** A component that manages and plays animation clips for a sprite. It keeps track of the current frame, playback time, and looping behavior.

### How to Code and Develop

1.  **Define Frame Data:** Specify the region (rectangle) of the sprite sheet that corresponds to each frame.
2.  **Animation Clip Class:** Create a class to store a list of frames, frame duration(s), and whether the animation should loop.
3.  **Animator Component:**
    *   Holds a reference to the current animation clip.
    *   Tracks the current playback time and current frame index.
    *   Updates the displayed frame of a sprite based on the elapsed time.
4.  **Integration with Sprite Renderer:** The Animator tells the Sprite Renderer which part of the sprite sheet (which frame) to draw by updating the `source_rect` of the `Sprite` object.

### Code Example (Sprite Sheet Animation)

```python
class AnimationFrame:
    def __init__(self, texture_source_rect, duration):
        # texture_source_rect: (x, y, width, height) on the sprite sheet
        self.texture_source_rect = texture_source_rect
        self.duration = duration # in seconds

class AnimationClip:
    def __init__(self, name, frames, loop=True):
        self.name = name
        self.frames = frames # List of AnimationFrame objects
        self.loop = loop
        self.total_duration = sum(frame.duration for frame in frames)
        if self.total_duration <= 0 and self.frames: # Handle zero duration animations
            self.total_duration = 0.0001 # a tiny duration to prevent division by zero

class Animator:
    def __init__(self, sprite_to_animate):
        self.sprite = sprite_to_animate # The Sprite object this animator controls
        self.animations = {} # Dictionary of AnimationClip objects, keyed by name
        self.current_animation_name = None
        self.current_animation_clip = None
        self.current_time = 0.0
        self.current_frame_index = 0
        self.is_playing = False

    def add_animation(self, clip):
        self.animations[clip.name] = clip

    def play(self, animation_name):
        if animation_name in self.animations:
            new_animation_clip = self.animations[animation_name]
            if self.current_animation_name != animation_name or not self.is_playing:
                self.current_animation_name = animation_name
                self.current_animation_clip = new_animation_clip
                self.current_time = 0.0
                self.current_frame_index = 0
                self.is_playing = True
                if not self.current_animation_clip.frames: # No frames to play
                    self.is_playing = False
                    if self.sprite: self.sprite.source_rect = None # Or some default
                    return
                self._update_sprite_texture_rect() # Set initial frame
        else:
            print(f"Warning: Animation '{animation_name}' not found.")

    def stop(self):
        self.is_playing = False

    def update(self, dt):
        if not self.is_playing or not self.current_animation_clip or not self.current_animation_clip.frames:
            return

        self.current_time += dt

        # Determine if the animation cycle has completed
        if self.current_time >= self.current_animation_clip.total_duration:
            if self.current_animation_clip.loop:
                self.current_time = self.current_time % self.current_animation_clip.total_duration
                self.current_frame_index = 0 # Reset to start for loop
            else:
                self.is_playing = False
                # Stay on the last frame
                self.current_frame_index = len(self.current_animation_clip.frames) - 1
                self._update_sprite_texture_rect()
                return # Animation finished

        # Find the current frame based on accumulated time
        time_accumulator = 0.0
        found_frame = False
        for i, frame in enumerate(self.current_animation_clip.frames):
            time_accumulator += frame.duration
            if self.current_time < time_accumulator:
                if self.current_frame_index != i:
                    self.current_frame_index = i
                    self._update_sprite_texture_rect()
                found_frame = True
                break

        # If, due to large dt or other reasons, we skipped past all frame durations
        # and it's a looping animation, ensure we are on a valid frame (usually the last one before loop or first)
        if not found_frame and self.current_animation_clip.loop:
             # This can happen if total_duration is very small or dt is large
             # Default to first frame if time reset, or last if something went wrong
            self.current_frame_index = 0 # Or len(self.current_animation_clip.frames) -1 if time_accumulator was exceeded
            self._update_sprite_texture_rect()


    def _update_sprite_texture_rect(self):
        if self.current_animation_clip and self.sprite and \
           0 <= self.current_frame_index < len(self.current_animation_clip.frames):
            current_anim_frame = self.current_animation_clip.frames[self.current_frame_index]
            self.sprite.source_rect = current_anim_frame.texture_source_rect
            # print(f"Animator: Playing '{self.current_animation_name}', Frame {self.current_frame_index}, Rect {self.sprite.source_rect}")
        elif self.sprite:
             # If no valid animation/frame, maybe clear source_rect or set to a default
             # self.sprite.source_rect = None
             pass


# Example Usage:
# (Assumes Sprite class exists and has a `source_rect` attribute used by a SpriteRenderer)
# (Assumes Texture class exists)

# class Sprite: # Simplified for example
#     def __init__(self, texture, position=(0,0)):
#         self.texture = texture
#         self.position = position
#         self.source_rect = None # (x, y, w, h) on the texture
#     def __str__(self): return "Sprite"

# class Texture: # Simplified
#     def __init__(self, name): self.name = name
#     def __str__(self): return f"Texture({self.name})"

# player_sprite_sheet = Texture("player_sheet.png")
# some_sprite = Sprite(player_sprite_sheet)

# walk_frames_data = [
#     {"rect": (0, 0, 32, 32), "duration": 0.1},
#     {"rect": (32, 0, 32, 32), "duration": 0.1},
#     {"rect": (64, 0, 32, 32), "duration": 0.1},
#     {"rect": (96, 0, 32, 32), "duration": 0.1},
# ]
# walk_animation_frames = [AnimationFrame(f["rect"], f["duration"]) for f in walk_frames_data]
# walk_animation = AnimationClip(name="walk", frames=walk_animation_frames, loop=True)

# idle_frames_data = [
#     {"rect": (0, 32, 32, 32), "duration": 0.5},
#     {"rect": (32, 32, 32, 32), "duration": 0.5},
# ]
# idle_animation_frames = [AnimationFrame(f["rect"], f["duration"]) for f in idle_frames_data]
# idle_animation = AnimationClip(name="idle", frames=idle_animation_frames, loop=True)

# animator = Animator(sprite_to_animate=some_sprite)
# animator.add_animation(walk_animation)
# animator.add_animation(idle_animation)

# animator.play("walk")

# In game loop:
# dt = 1.0 / 60.0 # Delta time
# animator.update(dt)
# sprite_renderer.render_sprite(some_sprite) # SpriteRenderer uses sprite.source_rect
```

**Explanation:**

*   `AnimationFrame`: Defines a single frame's source rectangle on the sprite sheet and its display duration.
*   `AnimationClip`: A named collection of `AnimationFrame`s, defining a complete animation (e.g., "walk", "idle") and its looping behavior.
*   `Animator`: Manages playback. The `play` method starts an animation. The `update` method advances the animation time, determines the correct frame based on frame durations, and updates the `source_rect` property on its target `Sprite`. The `SpriteRenderer` then uses this `source_rect` to draw the correct portion of the sprite sheet.

---

## 4. Tilemap Renderer

### Conceptual Overview

A Tilemap Renderer is used to efficiently draw game levels composed of a grid of tiles. Each tile is a small, reusable image. Tilemaps are fundamental for 2D games like platformers, RPGs, and strategy games.

Key features:

*   **Grid-based:** Levels are defined as a 2D array of tile IDs.
*   **Tileset:** A texture (similar to a sprite sheet) containing all unique tile images.
*   **Layers:** Tilemaps often support multiple layers for depth effects (e.g., background, foreground, collision layer).
*   **Efficiency:** Drawing a large map by rendering individual sprites for each tile would be slow. Tilemap renderers often use techniques like mesh generation or optimized batching.

### How to Code and Develop

1.  **Define Tile Data:** Store information about each tile type, such as its source rectangle in the tileset and potentially collision properties.
2.  **Map Data Structure:** Use a 2D array (or list of lists) to store the tile ID for each cell in the map grid for each layer.
3.  **Tileset Loading:** Load the tileset texture and understand its tile dimensions.
4.  **Rendering Logic:**
    *   Iterate through the visible portion of the tilemap (camera culling is crucial for performance).
    *   For each visible tile in each layer, get its ID from the map data.
    *   Look up the tile's source rectangle in the tileset based on its ID.
    *   Draw that portion of the tileset texture at the correct grid position on the screen, adjusted by camera position.
5.  **Optimization (Mesh Generation):** For static tilemaps, you can pre-generate a single (or few) large mesh(es) representing the entire map. This mesh consists of quads, each textured with the appropriate tile. This is much faster to render than individual draw calls per tile, as it's a single (or few) draw call(s). Dynamic updates to the mesh can be more complex.

### Code Example (Basic Tilemap Rendering)

```python
class Tileset:
    def __init__(self, texture, tile_width, tile_height):
        self.texture = texture # The Texture object for the tileset image
        self.tile_width = int(tile_width)
        self.tile_height = int(tile_height)

        if not texture or self.tile_width <= 0 or self.tile_height <= 0:
            self.columns = 0
            self.rows = 0
            print("Warning: Tileset texture is invalid or tile dimensions are non-positive.")
            return

        self.columns = texture.width // self.tile_width
        self.rows = texture.height // self.tile_height

    def get_tile_source_rect(self, tile_id):
        if tile_id <= 0 or self.columns == 0: # Assuming 0 or negative is an empty tile, or invalid tileset
            return None
        # Tile IDs are often 1-indexed in map editors.
        # Convert to 0-indexed for calculation.
        idx = tile_id - 1

        col = idx % self.columns
        row = idx // self.columns

        if row >= self.rows: # tile_id is out of bounds for this tileset
            return None

        return (col * self.tile_width, row * self.tile_height, self.tile_width, self.tile_height)

class TilemapLayer:
    def __init__(self, map_data, tile_width, tile_height, name="default_layer", visible=True):
        # map_data: 2D list of tile IDs, e.g., [[1, 2, 1], [3, 0, 3]]
        self.map_data = map_data
        self.tile_width = int(tile_width)
        self.tile_height = int(tile_height)
        self.name = name
        self.visible = visible

        self.rows = len(map_data)
        self.cols = len(map_data[0]) if self.rows > 0 else 0

class TilemapRenderer:
    def __init__(self, graphics_api, tileset):
        self.graphics_api = graphics_api # Hypothetical graphics API
        self.tileset = tileset
        self.layers = [] # List of TilemapLayer objects, ordered by rendering

    def add_layer(self, layer):
        if not isinstance(layer, TilemapLayer):
            print("Error: Attempted to add non-TilemapLayer object to TilemapRenderer.")
            return
        self.layers.append(layer)

    def render(self, camera_x, camera_y):
        if not self.tileset or not self.tileset.texture:
            # print("TilemapRenderer: No valid tileset to render.")
            return

        # Basic culling: determine visible tile range (simplified)
        # Assumes graphics_api has screen_width/height attributes
        screen_width_in_tiles = (self.graphics_api.screen_width // self.tileset.tile_width) + 2
        screen_height_in_tiles = (self.graphics_api.screen_height // self.tileset.tile_height) + 2

        start_col_world = int(camera_x // self.tileset.tile_width)
        start_row_world = int(camera_y // self.tileset.tile_height)

        end_col_world = start_col_world + screen_width_in_tiles
        end_row_world = start_row_world + screen_height_in_tiles


        for layer in self.layers:
            if not layer.visible or not layer.map_data:
                continue

            # Clamp rendering to the actual layer dimensions
            render_start_row = max(0, start_row_world)
            render_end_row = min(layer.rows, end_row_world)
            render_start_col = max(0, start_col_world)
            render_end_col = min(layer.cols, end_col_world)

            for r in range(render_start_row, render_end_row):
                for c in range(render_start_col, render_end_col):
                    tile_id = 0
                    try:
                        tile_id = layer.map_data[r][c]
                    except IndexError:
                        # This can happen if map_data is not rectangular or culling logic is off
                        # print(f"Warning: Tile index out of bounds at layer {layer.name}, row {r}, col {c}")
                        continue

                    if tile_id > 0: # Skip empty tiles (assuming 0 or less is empty)
                        source_rect = self.tileset.get_tile_source_rect(tile_id)
                        if source_rect:
                            # Calculate tile's position on screen
                            screen_pos_x = c * self.tileset.tile_width - camera_x
                            screen_pos_y = r * self.tileset.tile_height - camera_y

                            # The graphics_api.draw_texture_part would take the main texture,
                            # the destination rect on screen (x, y, w, h), and the source_rect from the texture.
                            self.graphics_api.draw_texture_part(
                                self.tileset.texture,
                                (screen_pos_x, screen_pos_y, self.tileset.tile_width, self.tileset.tile_height), # Destination rect on screen
                                source_rect  # Source rect from tileset (x, y, w, h)
                            )

# Example Usage (Conceptual - requires a graphics_api with screen_width/height and draw_texture_part)
# class Texture: # Simplified
#     def __init__(self, name, width=0, height=0):
#         self.name = name
#         self.image_data = f"Image data for {name}"
#         self.width = width
#         self.height = height

# class HypotheticalGraphicsAPI:
#     def __init__(self):
#         self.screen_width = 800
#         self.screen_height = 600
#     def draw_texture_part(self, texture, dest_rect_on_screen, source_rect_from_texture):
#         # dest_rect_on_screen: (screen_x, screen_y, width, height)
#         # source_rect_from_texture: (tex_x, tex_y, tex_width, tex_height)
#         print(f"Drawing part of '{texture.image_data}' from {source_rect_from_texture} to screen at {dest_rect_on_screen}")

# # Assume tileset.png is 64x32, containing two 32x32 tiles side-by-side.
# tileset_texture = Texture("tileset.png", width=64, height=32)

# my_tileset = Tileset(tileset_texture, tile_width=32, tile_height=32)

# # Tile ID 1 is the first tile (0,0) in tileset.png, Tile ID 2 is the second (32,0)
# map_level_1_data = [
#     [1, 1, 1, 1, 2, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1], # Example of a wider map
#     [1, 0, 0, 0, 2, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1],
#     [1, 0, 0, 0, 2, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1],
#     [1, 2, 2, 2, 2, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1],
# ]
# layer1 = TilemapLayer(map_level_1_data, my_tileset.tile_width, my_tileset.tile_height, name="Foreground")

# graphics = HypotheticalGraphicsAPI()
# tilemap_renderer = TilemapRenderer(graphics, my_tileset)
# tilemap_renderer.add_layer(layer1)

# In game loop:
# camera_x_world, camera_y_world = 0, 0 # Top-left of the camera's view in world coordinates
# tilemap_renderer.render(camera_x_world, camera_y_world)
# After a few seconds, move camera:
# camera_x_world, camera_y_world = 32, 0
# tilemap_renderer.render(camera_x_world, camera_y_world)
```

**Explanation:**

*   `Tileset`: Manages the tileset texture and provides `get_tile_source_rect(tile_id)` to find a tile's image on the sheet. It now includes basic validation.
*   `TilemapLayer`: Holds the 2D grid of tile IDs for a single layer, along with its visibility and name.
*   `TilemapRenderer`: Iterates through its `layers`. For each visible layer, it calculates which tiles are within the camera's view (culling). For each such tile, it gets its source rectangle from the `Tileset` and instructs the `graphics_api` to draw that specific part of the tileset texture at the calculated screen position.
*   `camera_x`, `camera_y`: Represent the top-left coordinates of the camera's viewport in world space.
*   The `graphics_api.draw_texture_part` method is key: it draws a specific rectangular portion of a source texture to a specific rectangular area on the screen.

---

## 5. Collision Handling

### Conceptual Overview

Collision Handling involves two main parts:

1.  **Collision Detection:** Determining if, when, and where two or more game objects (colliders) intersect.
2.  **Collision Response:** Deciding how objects react physically and logically to a collision.

Common collider shapes in 2D:

*   **Axis-Aligned Bounding Box (AABB):** Rectangles whose sides are aligned with the X and Y axes. Simple and fast for detection.
*   **Circle (Sphere in 3D):** Defined by a center point and radius.
*   **Polygon:** More complex shapes defined by a series of vertices.
*   **Pixel-Perfect:** Comparing actual sprite pixels (can be slow, used sparingly or as a final check after broader phase detection).

### How to Code and Develop

1.  **Collider Components:** Attach collider components (e.g., `AABBCollider`, `CircleCollider`) to game objects. These components define the shape and size of the object's collision bounds and usually hold a reference to their owner.
2.  **Collision Detection Algorithms:**
    *   **AABB vs. AABB:**
        ```
        bool checkAABB(rectA, rectB) {
            return rectA.x < rectB.x + rectB.width &&
                   rectA.x + rectA.width > rectB.x &&
                   rectA.y < rectB.y + rectB.height &&
                   rectA.y + rectA.height > rectB.y;
        }
        ```
    *   **Circle vs. Circle:**
        ```
        bool checkCircle(circleA, circleB) {
            float dx = circleA.center.x - circleB.center.x;
            float dy = circleA.center.y - circleB.center.y;
            float distanceSq = dx*dx + dy*dy;
            float radiiSum = circleA.radius + circleB.radius;
            return distanceSq < radiiSum * radiiSum;
        }
        ```
    *   More complex shapes often use algorithms like the Separating Axis Theorem (SAT).
3.  **Spatial Partitioning (Optimization):** For games with many objects, checking every pair of objects for collision (O(n^2)) is too slow. Spatial partitioning techniques (e.g., Quadtrees, Grids, Spatial Hashing) divide the game world into regions to quickly narrow down potential collision pairs.
4.  **Collision Response:**
    *   **Physical:** If using a physics engine, this often involves calculating impulse forces to make objects bounce or stop, and resolving interpenetration (moving objects apart so they no longer overlap).
    *   **Logical:** Trigger game events (e.g., player takes damage, picks up item, door opens). This is often handled via an event system or callbacks.
5.  **Event System:** When a collision is detected, an event can be dispatched (e.g., `OnCollisionEnter`, `OnCollisionStay`, `OnCollisionExit`) that other game systems or the involved objects can listen and react to.

### Code Example (Simple AABB Collision Detection and Event-like Callbacks)

```python
class AABBCollider:
    def __init__(self, x, y, width, height, owner=None, is_static=False, tag=None):
        self.x = x
        self.y = y
        self.width = width
        self.height = height
        self.owner = owner # The game object this collider belongs to
        self.is_static = is_static # Static colliders don't move or get resolved
        self.tag = tag # For identifying types of colliders (e.g., "player", "enemy", "bullet")

    def update_position(self, x, y):
        self.x = x
        self.y = y

    def get_rect(self):
        return (self.x, self.y, self.width, self.height)

    def intersects(self, other_collider):
        if not isinstance(other_collider, AABBCollider):
            # Could add checks for other collider types here (e.g., CircleCollider)
            return False

        rect_a = self.get_rect()
        rect_b = other_collider.get_rect()

        # AABB intersection check
        return (rect_a[0] < rect_b[0] + rect_b[2] and # rectA.x < rectB.x + rectB.width
                rect_a[0] + rect_a[2] > rect_b[0] and # rectA.x + rectA.width > rectB.x
                rect_a[1] < rect_b[1] + rect_b[3] and # rectA.y < rectB.y + rectB.height
                rect_a[1] + rect_a[3] > rect_b[1])    # rectA.y + rectA.height > rectB.y

class CollisionSystem:
    def __init__(self):
        self.colliders = []
        # For tracking ongoing collisions to manage OnCollisionEnter/Stay/Exit
        self.active_collision_pairs = set() # Stores frozensets of (id(collider_a), id(collider_b))

    def add_collider(self, collider):
        if collider not in self.colliders:
            self.colliders.append(collider)

    def remove_collider(self, collider):
        if collider in self.colliders:
            self.colliders.remove(collider)
        # Clean up from active_collision_pairs if it was involved
        pairs_to_remove = set()
        for pair_ids in self.active_collision_pairs:
            if id(collider) in pair_ids:
                pairs_to_remove.add(pair_ids)
        self.active_collision_pairs.difference_update(pairs_to_remove)


    def check_collisions(self):
        # Naive N^2 check. For many objects, use spatial partitioning (Quadtree, etc.)
        current_frame_collision_pairs = set()

        for i in range(len(self.colliders)):
            for j in range(i + 1, len(self.colliders)):
                collider_a = self.colliders[i]
                collider_b = self.colliders[j]

                # Skip check if both are static
                if collider_a.is_static and collider_b.is_static:
                    continue

                if collider_a.intersects(collider_b):
                    # Use frozenset of object ids to make the pair order-independent and hashable
                    collision_pair_ids = frozenset((id(collider_a), id(collider_b)))
                    current_frame_collision_pairs.add(collision_pair_ids)

                    if collision_pair_ids not in self.active_collision_pairs:
                        # New collision: OnCollisionEnter
                        self._dispatch_event(collider_a, collider_b, "on_collision_enter")
                        self._dispatch_event(collider_b, collider_a, "on_collision_enter")
                    else:
                        # Ongoing collision: OnCollisionStay
                        self._dispatch_event(collider_a, collider_b, "on_collision_stay")
                        self._dispatch_event(collider_b, collider_a, "on_collision_stay")

        # Check for collisions that ended (OnCollisionExit)
        exited_collision_pairs = self.active_collision_pairs - current_frame_collision_pairs
        for pair_ids in exited_collision_pairs:
            # Need to retrieve the actual collider objects from their IDs or store them differently.
            # This part is tricky if colliders can be removed. A safer way is to store actual collider pairs.
            # For simplicity, this example assumes colliders are persistent enough for this frame.
            # A more robust solution might involve looking up colliders by ID from a central registry.
            # This is a known challenge in such event systems.
            # Let's assume we can find them for the event dispatch (conceptual)
            # This part requires a way to get colliders from IDs stored in pair_ids, which is not directly implemented here.
            # For a real system, you might store (collider_a, collider_b) tuples directly if performance allows,
            # or use a mapping from ID back to object.
            # Simplified: if you stored actual collider objects in pairs, you'd use those.
            # For this example, we'll just acknowledge the event conceptually.
            # print(f"Exit event for pair {pair_ids} - actual objects needed for dispatch")
            # To properly dispatch exit, you'd need to find the colliders that correspond to pair_ids
            # This is a simplification:
            # Find colliders involved in the exited pair to dispatch events to their owners
            # This lookup is not robust if colliders are frequently added/removed.
            collider_a_exit, collider_b_exit = None, None
            id_list = list(pair_ids)
            for c in self.colliders: # This is inefficient; better to have a map id->collider
                if id(c) == id_list[0]: collider_a_exit = c
                elif id(c) == id_list[1]: collider_b_exit = c
                if collider_a_exit and collider_b_exit: break

            if collider_a_exit and collider_b_exit:
                 self._dispatch_event(collider_a_exit, collider_b_exit, "on_collision_exit")
                 self._dispatch_event(collider_b_exit, collider_a_exit, "on_collision_exit")


        self.active_collision_pairs = current_frame_collision_pairs

    def _dispatch_event(self, collider_1, collider_2, event_method_name):
        if collider_1.owner and hasattr(collider_1.owner, event_method_name):
            method = getattr(collider_1.owner, event_method_name)
            method(collider_2) # Pass the other collider involved

# Example Usage:
# class GameObject: # Base class for things that can collide
#     def __init__(self, name, x, y, w, h, is_static=False, tag=None):
#         self.name = name
#         self.x, self.y = x, y
#         self.collider = AABBCollider(x, y, w, h, owner=self, is_static=is_static, tag=tag)

#     def update_position(self, new_x, new_y): # Call this when game object moves
#         self.x, self.y = new_x, new_y
#         self.collider.update_position(new_x, new_y)

#     def on_collision_enter(self, other_collider):
#         print(f"ENTER: {self.name} (tag: {self.collider.tag}) collided with {other_collider.owner.name} (tag: {other_collider.tag})")

#     def on_collision_stay(self, other_collider):
#         # print(f"STAY: {self.name} is still colliding with {other_collider.owner.name}")
#         pass

#     def on_collision_exit(self, other_collider):
#         print(f"EXIT: {self.name} no longer colliding with {other_collider.owner.name}")

# player = GameObject("Player", 50, 50, 32, 32, tag="player")
# wall = GameObject("Wall", 100, 50, 20, 100, is_static=True, tag="environment")
# enemy = GameObject("Enemy", 60, 60, 24, 24, tag="enemy")

# collision_sys = CollisionSystem()
# collision_sys.add_collider(player.collider)
# collision_sys.add_collider(wall.collider)
# collision_sys.add_collider(enemy.collider)

# In game loop (conceptual):
# frame_count = 0
# while frame_count < 5:
#     print(f"\n--- Frame {frame_count} ---")
#     # Simulate player movement causing different collision states
#     if frame_count == 0: # Initial state, player might be colliding with enemy
#         player.update_position(player.x, player.y) # No move, just check initial
#     if frame_count == 1: # Player moves into wall
#         player.update_position(90, 50)
#     elif frame_count == 2: # Player stays in wall
#         player.update_position(90, 50)
#     elif frame_count == 3: # Player moves away from wall and enemy
#         player.update_position(10, 10)
#     elif frame_count == 4: # Player stays away
#         player.update_position(10,10)

#     # Update enemy position if it moves
#     # enemy.update_position(enemy.x + 1, enemy.y)

#     collision_sys.check_collisions()
#     frame_count += 1
```

**Explanation:**

*   `AABBCollider`: Represents an axis-aligned bounding box. It stores its dimensions, position (which should be updated when its owner moves), a reference to its `owner` game object, an `is_static` flag, and a `tag`. The `intersects` method checks for overlap with another `AABBCollider`.
*   `CollisionSystem`:
    *   `colliders`: A list of all active colliders in the scene.
    *   `active_collision_pairs`: A set used to track which pairs of colliders were intersecting in the *previous* frame. This is essential for distinguishing between `OnCollisionEnter` (was not colliding, now is), `OnCollisionStay` (was colliding, still is), and `OnCollisionExit` (was colliding, now is not).
    *   `check_collisions()`:
        *   Iterates through all unique pairs of colliders (naive O(n^2) approach).
        *   Skips static-vs-static checks.
        *   If an intersection occurs:
            *   It creates a `collision_pair_ids` (a `frozenset` of the `id()`s of the two colliders, making the pair order-independent and hashable for set operations).
            *   This pair is added to `current_frame_collision_pairs`.
            *   If this pair was *not* in `active_collision_pairs` (from the previous frame), it's an `OnCollisionEnter` event.
            *   If it *was* in `active_collision_pairs`, it's an `OnCollisionStay` event.
        *   After checking all pairs, `exited_collision_pairs` are determined by finding pairs in `active_collision_pairs` that are *not* in `current_frame_collision_pairs`. These trigger `OnCollisionExit`.
        *   Finally, `active_collision_pairs` is updated to `current_frame_collision_pairs` for the next frame's comparison.
    *   `_dispatch_event()`: Calls a specific method (e.g., `on_collision_enter`) on the `owner` of the collider, passing the *other* collider involved in the collision.
*   **Important:** For physics-based responses (like stopping movement or bouncing), this detection should ideally occur *before* an object visually moves into another, or you need penetration resolution algorithms. This example focuses on the detection and event dispatching logic. The exit event dispatch has known complexities regarding retrieving objects from IDs if objects are frequently destroyed.

---

## 6. Audio Player

### Conceptual Overview

An Audio Player system handles the playback of sound effects (SFX) and background music (BGM) in a game.

Key features:

*   **Loading Audio Files:** Supports various audio formats (e.g., WAV for uncompressed SFX, MP3/OGG for compressed BGM).
*   **Playing Sounds:** Ability to play sounds once, loop them, or play multiple sounds simultaneously (polyphony).
*   **Volume Control:** Global volume, per-sound volume, and potentially per-category (SFX, Music, UI) volume.
*   **Panning:** Positioning sounds in a stereo (or 3D) soundscape (more advanced).
*   **Streaming:** For long audio files like BGM, stream them from disk rather than loading the entire file into memory.
*   **Channels/Sources:** Manage available audio channels to play multiple sounds.

### How to Code and Develop

1.  **Choose an Audio Library:** Strongly recommended. Examples include:
    *   **Python:** `pygame.mixer`, `playsound` (simple), `python-sounddevice` + `soundfile` (more control), `pyglet.media`.
    *   **C++/Others:** SDL_mixer, OpenAL, FMOD, Wwise, SoLoud.
    *   **Web:** Web Audio API.
    Reinventing audio playback from scratch is very complex.
2.  **Sound Resource Management:** Create a class to load and cache sound data. `Sound` objects might wrap the library's sound objects.
3.  **Audio Manager/Player Class:**
    *   Provides a simple API: `play_sfx(sound_name)`, `play_music(music_name)`, `stop_music()`, `set_volume()`.
    *   Manages loaded sound resources.
    *   Interacts with the chosen audio library to control playback, channels, volume, looping, etc.
4.  **Error Handling:** Gracefully handle cases where audio files are missing or the audio device has issues.

### Code Example (Conceptual - using a hypothetical Python audio library similar to `pygame.mixer`)

```python
# This example is highly conceptual and mimics patterns seen in libraries like Pygame Mixer.
# Actual implementation details depend heavily on the chosen audio library.

class SoundData: # Represents a loaded sound effect or music track
    def __init__(self, filepath, sound_object_from_lib):
        self.filepath = filepath
        self.lib_sound_obj = sound_object_from_lib # The actual sound object from the audio library

class AudioPlayer:
    def __init__(self, audio_library_wrapper):
        self.audio_lib = audio_library_wrapper # Your wrapper around the actual audio library
        self.sfx_cache = {}  # sound_id -> SoundData
        self.music_cache = {} # music_id -> SoundData

        self.current_music_id = None
        self._global_sfx_volume = 1.0 # 0.0 to 1.0
        self._global_music_volume = 1.0 # 0.0 to 1.0

        self.audio_lib.init_mixer() # Initialize the underlying audio system

    def load_sfx(self, sfx_id, filepath):
        if sfx_id not in self.sfx_cache:
            sound_obj = self.audio_lib.load_sound_effect(filepath)
            if sound_obj:
                self.sfx_cache[sfx_id] = SoundData(filepath, sound_obj)
                print(f"SFX '{sfx_id}' loaded from '{filepath}'")
            else:
                print(f"Warning: Failed to load SFX '{sfx_id}' from '{filepath}'")
                self.sfx_cache[sfx_id] = None # Mark as failed
        return self.sfx_cache.get(sfx_id)

    def load_music(self, music_id, filepath):
        if music_id not in self.music_cache:
            # Music is often handled differently (streaming) by libraries
            # For this example, assume load_music_track prepares it for playing/streaming
            music_obj_ref = self.audio_lib.load_music_track(filepath) # Might just be filepath for streaming libs
            if music_obj_ref: # Could be the path itself or a library object
                self.music_cache[music_id] = SoundData(filepath, music_obj_ref)
                print(f"Music '{music_id}' loaded/prepared from '{filepath}'")
            else:
                print(f"Warning: Failed to load Music '{music_id}' from '{filepath}'")
                self.music_cache[music_id] = None
        return self.music_cache.get(music_id)

    def play_sfx(self, sfx_id, volume_multiplier=1.0, loops=0):
        sound_data = self.sfx_cache.get(sfx_id)
        if sound_data and sound_data.lib_sound_obj:
            effective_volume = self._global_sfx_volume * volume_multiplier
            self.audio_lib.play_sfx_on_channel(sound_data.lib_sound_obj, effective_volume, loops)
            # print(f"Playing SFX: {sfx_id} at volume {effective_volume:.2f}")
        else:
            print(f"SFX '{sfx_id}' not found or not loaded.")

    def play_music(self, music_id, volume_multiplier=1.0, loops=-1, fade_ms=0): # loops = -1 for infinite
        music_data = self.music_cache.get(music_id)
        if music_data and music_data.lib_sound_obj: # lib_sound_obj might be the filepath for music
            effective_volume = self._global_music_volume * volume_multiplier
            self.audio_lib.play_music_track(music_data.lib_sound_obj, effective_volume, loops, fade_ms)
            self.current_music_id = music_id
            # print(f"Playing Music: {music_id} at volume {effective_volume:.2f}")
        else:
            print(f"Music '{music_id}' not found or not loaded.")

    def stop_music(self, fade_ms=0):
        self.audio_lib.stop_music_playback(fade_ms)
        self.current_music_id = None
        # print("Music stopped.")

    def pause_music(self):
        self.audio_lib.pause_music_playback()

    def resume_music(self):
        self.audio_lib.resume_music_playback()

    def set_sfx_volume(self, volume): # Sets global SFX volume
        self._global_sfx_volume = max(0.0, min(1.0, volume))
        # Some libraries might allow adjusting all SFX channels, others don't directly
        print(f"Global SFX volume set to {self._global_sfx_volume:.2f}")

    def set_music_volume(self, volume): # Sets global Music volume
        self._global_music_volume = max(0.0, min(1.0, volume))
        if self.current_music_id: # If music is playing, adjust its volume
            self.audio_lib.set_current_music_playback_volume(self._global_music_volume)
        print(f"Global Music volume set to {self._global_music_volume:.2f}")

    def is_music_playing(self):
        return self.audio_lib.get_is_music_playing()

# --- Conceptual Audio Library Wrapper (Interface for your chosen library) ---
# class MyAudioLibWrapper:
#     def __init__(self): self.is_initialized = False
#     def init_mixer(self):
#         # e.g., pygame.mixer.init(frequency=44100, size=-16, channels=2, buffer=512)
#         self.is_initialized = True
#         print("Audio Library Initialized (Conceptual).")
#     def load_sound_effect(self, filepath):
#         # e.g., return pygame.mixer.Sound(filepath)
#         if not self.is_initialized: return None
#         print(f"LIB: Loading SFX {filepath}")
#         return {"path": filepath, "type": "sfx"} # Placeholder
#     def load_music_track(self, filepath):
#         # For pygame.mixer.music, you just need the path. It's loaded on play.
#         if not self.is_initialized: return None
#         print(f"LIB: Preparing Music track {filepath}")
#         return filepath # Placeholder, actual object might not be needed until play
#     def play_sfx_on_channel(self, sfx_obj, volume, loops):
#         # e.g., channel = sfx_obj_pygame.play(loops=loops)
#         # channel.set_volume(volume)
#         if not self.is_initialized: return
#         print(f"LIB: Playing SFX {sfx_obj['path']} at vol {volume}, loops: {loops}")
#     def play_music_track(self, music_obj_ref, volume, loops, fade_ms): # music_obj_ref is often filepath
#         # e.g., pygame.mixer.music.load(music_obj_ref)
#         # pygame.mixer.music.set_volume(volume)
#         # pygame.mixer.music.play(loops=loops, fade_ms=fade_ms)
#         if not self.is_initialized: return
#         print(f"LIB: Playing Music {music_obj_ref} at vol {volume}, loops: {loops}, fade: {fade_ms}ms")
#     def stop_music_playback(self, fade_ms):
#         # e.g., pygame.mixer.music.fadeout(fade_ms) or pygame.mixer.music.stop()
#         if not self.is_initialized: return
#         print(f"LIB: Stopping music (fade: {fade_ms}ms)")
#     def pause_music_playback(self):
#         # e.g., pygame.mixer.music.pause()
#         if not self.is_initialized: return
#         print("LIB: Pausing music")
#     def resume_music_playback(self):
#         # e.g., pygame.mixer.music.unpause()
#         if not self.is_initialized: return
#         print("LIB: Resuming music")
#     def set_current_music_playback_volume(self, volume):
#         # e.g., pygame.mixer.music.set_volume(volume)
#         if not self.is_initialized: return
#         print(f"LIB: Setting current music volume to {volume}")
#     def get_is_music_playing(self):
#         # e.g., return pygame.mixer.music.get_busy()
#         if not self.is_initialized: return False
#         return True # Placeholder

# Example Usage:
# my_audio_lib_wrapper = MyAudioLibWrapper()
# audio_player = AudioPlayer(my_audio_lib_wrapper)

# audio_player.load_sfx("jump", "sfx/jump.wav")
# audio_player.load_sfx("coin", "sfx/coin.ogg") # OGG is also good for SFX
# audio_player.load_music("level_theme", "music/level1.mp3")

# audio_player.set_sfx_volume(0.8)
# audio_player.set_music_volume(0.5)

# audio_player.play_music("level_theme")
# # In game, when player jumps:
# # audio_player.play_sfx("jump", volume_multiplier=0.9) # 0.9 * global_sfx_volume
# # When player collects coin:
# # audio_player.play_sfx("coin")
```

**Explanation:**

*   `SoundData`: A simple container for a loaded sound/music resource and its original filepath.
*   `AudioPlayer`:
    *   Takes an `audio_library_wrapper` in its constructor. This wrapper would contain the actual calls to your chosen audio library (e.g., `pygame.mixer` functions).
    *   `sfx_cache` and `music_cache`: Dictionaries to store loaded `SoundData` objects, preventing redundant loading.
    *   `load_sfx`/`load_music`: Load audio files via the wrapper and store them.
    *   `play_sfx`/`play_music`: Play sounds using the wrapper. They apply global volume settings and per-call volume multipliers. Music playback often supports looping and fading.
    *   Volume control (`set_sfx_volume`, `set_music_volume`): Adjusts the base volume for categories.
    *   `stop_music`, `pause_music`, `resume_music`: Basic music playback controls.
*   **`MyAudioLibWrapper` (Conceptual):** This class is crucial. It's an abstraction layer. You'd implement its methods using the specific functions of your chosen audio library. This keeps `AudioPlayer` cleaner and makes it easier to swap audio libraries if needed.
*   **Error Handling:** The example includes basic print statements for errors. A robust system would have more sophisticated error logging or fallback mechanisms.

---

## 7. YAML Serialization

### Conceptual Overview

Serialization is the process of converting an object's state (its data) into a format that can be stored (e.g., in a file) or transmitted. Deserialization is the reverse process: reconstructing the object from that stored format.

YAML (YAML Ain't Markup Language) is a human-readable data serialization language. It's often used for configuration files, data exchange, and saving game state or level data because of its clarity and ease of use compared to formats like JSON (for complex, nested data) or XML.

Common use cases in game development:

*   **Configuration Files:** Game settings (resolution, audio volumes, key bindings), entity archetypes, item definitions.
*   **Level Data:** Storing object placements, tilemap layouts, NPC properties, event triggers.
*   **Game Saves:** Saving player progress, inventory, current world state (though binary formats might be preferred for performance/security for save games).

### How to Code and Develop

1.  **Choose a YAML Library:** Most programming languages have well-supported YAML libraries (e.g., `PyYAML` for Python, `SnakeYAML` for Java, `yaml-cpp` for C++, `serde_yaml` for Rust, `YAMLDotNet` for C#).
2.  **Define Data Structures:** The data you want to serialize/deserialize should be organized into classes or structures. For simple data, built-in types like dictionaries and lists work fine. For complex game objects, custom classes are better.
3.  **Serialization (Saving to YAML):**
    *   Populate your data structures with the current game state or configuration.
    *   Use the YAML library's "dump" or "serialize" function to convert these structures into a YAML formatted string or write directly to a file.
4.  **Deserialization (Loading from YAML):**
    *   Use the YAML library's "load" or "deserialize" function to read data from a YAML file or string.
    *   The library will typically reconstruct your objects or provide dictionaries/lists that you then use to populate your game's data structures. **Always use "safe load" functions if available** to prevent arbitrary code execution from untrusted YAML sources.
5.  **Custom Type Handling (Advanced):**
    *   To make YAML work seamlessly with your custom game classes (e.g., `Enemy`, `Item`, `Vector2D`), you often need to tell the YAML library how to represent these classes in YAML and how to reconstruct them.
    *   This usually involves registering "representers" (object to YAML) and "constructors" (YAML to object) for your custom types.

### Code Example (Python with PyYAML)

First, ensure you have PyYAML installed: `pip install PyYAML`

```python
import yaml

# --- 1. Basic Data Serialization (Dictionaries, Lists) ---
game_settings = {
    "game_title": "Adventures in YAML",
    "version": "1.0.2",
    "display": {
        "width": 1920,
        "height": 1080,
        "fullscreen": True,
        "vsync": False
    },
    "audio": {
        "master_volume": 75, # Percentage
        "music_volume": 60,
        "sfx_volume": 80
    },
    "player_defaults": {
        "start_health": 100,
        "start_mana": 50,
        "inventory_size": 20
    },
    "enemy_types_enabled": ["goblin", "orc", "skeleton"]
}

# Serialize to YAML string
try:
    yaml_string_output = yaml.dump(game_settings, sort_keys=False, indent=2)
    print("--- Serialized YAML String ---")
    print(yaml_string_output)
except yaml.YAMLError as e:
    print(f"Error during YAML dumping: {e}")


# Serialize to a file
settings_filepath = "game_settings.yaml"
try:
    with open(settings_filepath, "w") as f:
        yaml.dump(game_settings, f, sort_keys=False, indent=2, Dumper=yaml.SafeDumper) # Use SafeDumper for output if desired
    print(f"Saved settings to {settings_filepath}")
except Exception as e:
    print(f"Error saving YAML to file: {e}")


# Deserialize from YAML string
try:
    # Always use safe_load for untrusted or even trusted sources as a best practice
    loaded_settings_from_string = yaml.safe_load(yaml_string_output)
    print("\n--- Deserialized from String ---")
    if loaded_settings_from_string:
        print(f"Game Title (from string): {loaded_settings_from_string.get('game_title')}")
        print(f"Master Volume (from string): {loaded_settings_from_string.get('audio', {}).get('master_volume')}")
except yaml.YAMLError as e:
    print(f"Error parsing YAML string: {e}")


# Deserialize from a file
try:
    with open(settings_filepath, "r") as f:
        loaded_settings_from_file = yaml.safe_load(f)
    print("\n--- Deserialized from File ---")
    if loaded_settings_from_file:
        print(f"Fullscreen (from file): {loaded_settings_from_file.get('display', {}).get('fullscreen')}")
        print(f"Enabled enemies (from file): {loaded_settings_from_file.get('enemy_types_enabled')}")
except FileNotFoundError:
    print(f"Error: {settings_filepath} not found.")
except yaml.YAMLError as e:
    print(f"Error parsing {settings_filepath}: {e}")
except Exception as e:
    print(f"Error loading YAML from file: {e}")


# --- 2. Serializing Custom Objects ---
class Vec2D: # A simple custom class
    yaml_tag = '!Vec2D' # Recommended convention for custom tags

    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __repr__(self):
        return f"Vec2D(x={self.x}, y={self.y})"

    # Representer: How to convert Vec2D object to YAML node
    @classmethod
    def to_yaml(cls, representer, node):
        # Represent as a sequence (list) in YAML: [x, y]
        return representer.represent_sequence(cls.yaml_tag, [node.x, node.y], flow_style=True)
        # Alternatively, represent as a mapping (dictionary):
        # return representer.represent_mapping(cls.yaml_tag, {'x': node.x, 'y': node.y})


    # Constructor: How to create Vec2D object from YAML node
    @classmethod
    def from_yaml(cls, constructor, node):
        # If represented as a sequence:
        values = constructor.construct_sequence(node, deep=True)
        return cls(values[0], values[1])
        # If represented as a mapping:
        # mapping = constructor.construct_mapping(node, deep=True)
        # return cls(mapping['x'], mapping['y'])

# Register the custom class with PyYAML's SafeLoader/SafeDumper
yaml.add_representer(Vec2D, Vec2D.to_yaml, Dumper=yaml.SafeDumper)
yaml.add_constructor(Vec2D.yaml_tag, Vec2D.from_yaml, Loader=yaml.SafeLoader)


class GameObjectData:
    yaml_tag = '!GameObject'

    def __init__(self, name, position, health, tags=None):
        self.name = name
        self.position = position # Expected to be a Vec2D instance
        self.health = health
        self.tags = tags if tags is not None else []

    def __repr__(self):
        return (f"GameObjectData(name='{self.name}', position={self.position}, "
                f"health={self.health}, tags={self.tags})")

    @classmethod
    def to_yaml(cls, representer, node):
        return representer.represent_mapping(cls.yaml_tag, {
            'name': node.name,
            'position': node.position, # PyYAML will use Vec2D's representer
            'health': node.health,
            'tags': node.tags
        })

    @classmethod
    def from_yaml(cls, constructor, node):
        mapping = constructor.construct_mapping(node, deep=True)
        return cls(
            mapping['name'],
            mapping['position'], # PyYAML will use Vec2D's constructor
            mapping['health'],
            mapping.get('tags', [])
        )

yaml.add_representer(GameObjectData, GameObjectData.to_yaml, Dumper=yaml.SafeDumper)
yaml.add_constructor(GameObjectData.yaml_tag, GameObjectData.from_yaml, Loader=yaml.SafeLoader)


# Create instances with custom objects
player_obj = GameObjectData(name="Hero", position=Vec2D(10, 25.5), health=100, tags=["player", "主角"])
enemy_obj = GameObjectData(name="Goblin", position=Vec2D(50, 40.2), health=30, tags=["enemy", "hostile"])
level_objects = {"scene_name": "Forest Clearing", "objects": [player_obj, enemy_obj]}


# Serialize data with custom objects
level_filepath = "level_data.yaml"
try:
    with open(level_filepath, "w", encoding="utf-8") as f: # Use utf-8 for non-ASCII tags
        yaml.dump(level_objects, f, sort_keys=False, indent=2, Dumper=yaml.SafeDumper, allow_unicode=True)
    print(f"\nSaved level data with custom objects to {level_filepath}")
except Exception as e:
    print(f"Error saving level data: {e}")

# Deserialize data with custom objects
try:
    with open(level_filepath, "r", encoding="utf-8") as f:
        loaded_level_data = yaml.safe_load(f)
    print("\n--- Deserialized Custom Objects from File ---")
    if loaded_level_data:
        print(f"Scene: {loaded_level_data.get('scene_name')}")
        for obj_data in loaded_level_data.get('objects', []):
            print(f"  Loaded: {obj_data}")
            if isinstance(obj_data, GameObjectData) and isinstance(obj_data.position, Vec2D):
                print(f"    Position X: {obj_data.position.x}")
except FileNotFoundError:
    print(f"Error: {level_filepath} not found.")
except yaml.YAMLError as e:
    print(f"Error parsing {level_filepath}: {e}")
except Exception as e:
    print(f"Error loading level data: {e}")

```

**`game_settings.yaml` (created by the script):**
```yaml
game_title: Adventures in YAML
version: 1.0.2
display:
  width: 1920
  height: 1080
  fullscreen: true
  vsync: false
audio:
  master_volume: 75
  music_volume: 60
  sfx_volume: 80
player_defaults:
  start_health: 100
  start_mana: 50
  inventory_size: 20
enemy_types_enabled:
  - goblin
  - orc
  - skeleton
```

**`level_data.yaml` (created by the script):**
```yaml
scene_name: Forest Clearing
objects:
  - !GameObject
    name: Hero
    position: !Vec2D [10, 25.5]
    health: 100
    tags:
      - player
      - 主角
  - !GameObject
    name: Goblin
    position: !Vec2D [50, 40.2]
    health: 30
    tags:
      - enemy
      - hostile
```

**Explanation:**

*   **Basic Serialization:** `yaml.dump()` converts Python dictionaries, lists, and basic types into YAML format. `yaml.safe_load()` (very important!) parses YAML back into Python objects, preventing execution of arbitrary code that could be embedded in malicious YAML files. Using `indent=2` makes the YAML file human-readable. `sort_keys=False` maintains the order of keys from Python dictionaries (Python 3.7+).
*   **Custom Objects (`Vec2D`, `GameObjectData`):**
    *   `yaml_tag`: A unique string (conventionally starting with `!`) that PyYAML uses in the YAML file to identify the class of the object. This allows `safe_load` to know which constructor to call.
    *   `to_yaml` (representer): A class method registered with `yaml.add_representer()`. It tells PyYAML how to convert an instance of your custom class into a YAML representation (e.g., a sequence for `Vec2D` like `[x, y]`, or a mapping for `GameObjectData`).
    *   `from_yaml` (constructor): A class method registered with `yaml.add_constructor()`. It tells PyYAML how to create an instance of your custom class from the corresponding YAML data.
    *   `Dumper=yaml.SafeDumper` and `Loader=yaml.SafeLoader`: It's good practice to register custom types with the safe versions of the Dumper and Loader.
    *   `allow_unicode=True`: Useful if your string data might contain non-ASCII characters.
*   This system allows you to work with your native Python objects and easily save/load them to/from human-readable YAML files, which is excellent for configuration, level design, and simple game state.

---

This guide provides a foundational understanding of these game development components. Each topic can become significantly more complex depending on the requirements of your game. Remember to consult documentation for specific libraries or game engines you choose to use, as they often provide optimized and feature-rich implementations of these systems.

Happy Gamedev!
