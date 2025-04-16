## 📦 Endless Sky Systems — Detailed Combined README
This document provides an in-depth breakdown of Endless Sky's core game systems, covering their functionality, workflows, responsibilities, and data structures. It combines the internal designs of the Mission System, Ship System, Weapon System, Map Panel, and Trading Panel.
________________________________________
## 🌌 Mission System
Overview
The Mission system manages the entire lifecycle of missions in Endless Sky, handling how missions are generated, validated, assigned, progressed, and completed or failed.
Mission Lifecycle Process  

##	Instantiation

o	Missions are created from templates.  

o	Destination, waypoints, and stopovers are dynamically selected using filters.

o	Waypoints are chosen via LocationFilter::PickSystem().

o	Stopovers are added using stopoverFilters, ensuring they are valid.

o	If no destination is found, defaults to the player's current planet.

##	Validation and Conditions

  o	IsValid(): Ensures necessary locations and data are present.
  
  o	CanOffer(): Verifies if a mission should be offered now, checking player location, filters, repeat limits, and triggers.
  
  o	HasSpace(): Confirms the player has enough cargo/bunks.
  
  o	CanComplete() and IsSatisfied(): Validate mission completion conditions.
  
## Event Handling

o	The Mission::Do() function processes mission events.

o	Events include DESTROY, BOARD, DISABLE, and JUMP.

o	Affects mission state and triggers reactions for NPC ships linked to missions.

## Cargo & Passenger Failure Checks

o	Verifies whether mission-related cargo or passengers were lost or stolen.

o	Example:

	for (auto &it : ship->Cargo().MissionCargo())

    if (it.first == this)	 
    
    failed = true;
    
## Dynamic Location Selection

o	Locations are dynamically picked using LocationFilter::PickSystem() and PickPlanet().

o	Selection is based on proximity, properties, and clearance flags.

## Trade & Commodity Selection
o	Implements profit-weighted random selection for mission commodities.

o	Higher-profit commodities have increased probability.

## Data Structures
Purpose	Data Structure

Waypoints	std::set<const System *>

Stopovers	std::set<const Planet *>

Mission Cargo	std::map<const Mission *, Item>

Passenger List	std::map<const Mission *, bool>

Mission Triggers	std::map<Trigger, MissionAction>

Commodity Price Map	std::map<std::string, TradeInfo>

________________________________________

## 🚀 Ship System

### Overview

Covers the first 1000 lines of Ship.cpp, responsible for loading, parsing, and initializing ship definitions and attributes from data files.

Ship Loading Workflow

###	Definition Parsing

####	Ship::Load(const DataNode &node) reads YAML-like definitions.

####	Extracts:

	Names and display labels

	Visual elements (sprite, thumbnail)

	Crew, cargo, fuel, position, and special flags

####	Hardpoints, Bays, and Explosions

o	Engines, reverse engines, and steering engines parsed with zoom, angle, gimbal, and location.

o	Bays handle fighters/drones with launch angles and effects.

o	Leak effects and explosion visuals are configured.

####	Inheritance Handling

o	Ships inherit attributes and hardpoints from base models.

o	Deferred merging and validation occurs in FinishLoading().

####	Outfit Parsing and Validation

o	outfits map holds Outfit pointers and their counts.

o	Checks for mismatched weapons and equipment.

####	Final Loading and Calculations

o	Computes deterrence/attraction values.

o	Validates weapon and hardpoint configurations.

o	Merges missing and variant attributes.

### Data Structures

##### Purpose	   Data Structure

Weapons	  vector<Hardpoint>

Bays	   vector<Bay>

Leaks	   vector<Leak>

Outfits	   map<const Outfit *, int>

Effects	    map<const Effect *, int>

Attributes	  ShipInfo / Attributes

________________________________________

# 🔫 Weapon System

## Overview

Defines weapon configurations, effects, dropoff behavior, submunitions, visuals, and sound effects.

### Weapon Setup and Loading

####	LoadWeapon()

o	Reads weapon data from a DataNode.

o	Sets behavior flags (stream, cluster, phasing).

o	Loads sprite, sound, icon.

o	Configures effects for firing, hit, death.

o	Loads ammo types, submunitions.

o	Assigns numeric properties (damage, velocity, tracking, reload, dropoff).

o	Ensures consistency and applies defaults (hull damage fallback).

####	Damage and Dropoff Calculations

o	Computes shield, hull, ion damage.

o	Linear dropoff between min and max ranges.

o	Submunitions included in TotalDamage() calculations.

####	Getter Methods

o	IsWeapon(), Range(), Inaccuracy(), TotalLifetime().

o	Access to submunitions, visuals, ammo usage, and effects.

##### Data Structures

Purpose	Data Structure

Damage Values	Numeric fields (double, int)

Effects	Effect* pointers

Ammo & Submunitions	Outfit* pointers, usage values

Distribution	Distribution for inaccuracy

________________________________________

# 🛡️ MapPanel Module

## Overview

Handles the in-game star map, providing visual navigation, overlays, and system/planet search functionality.

## MapPanel Responsibilities

### Find()

o	Searches all known systems/planets by name.

o	Uses Format::Search() for matching.

o	Selects closest match, recenters the map.

###	Zoom()

o	Computes zoom factor as pow(1.5, player.MapZoom()).

###	IsSatisfied()

o	Checks if a mission's conditions are met in current game state.

###	GetTravelInfo()

o	Determines connection type: Hyperlink, Wormhole, Jump.

o	Returns mappability, color, flags.

###	CenterOnSystem()

o	Moves map view to a given system.

o	Uses instant or animated transition.

##	UpdateCache()

o	Caches system positions, links, and colors.

o	Optimizes map rendering performance.

## Data Structures

### Purpose	Data Structure

Star Systems	unordered_map<string, System>

Planets	unordered_map<string, Planet>

Stellar Objects	vector<StellarObject>

Map Nodes	vector<MapNode>

Map Links	vector<MapLink>

Colors & Flags	Color, bool, string

________________________________________

# 💸 TradingPanel Module

## Overview

Controls the commodity trading UI, allowing players to buy, sell, and track cargo and trade profits.

## TradingPanel Workflow

### 	TradingPanel()

o	Initializes system commodity prices, player cargo, and commodity list.

### 	~TradingPanel()

o	Displays a profit/loss summary message for the session.

### 	Step()

o	Updates tooltips and user prompts.

### 	Draw()

o	Renders the trading UI, commodity list, prices, profits, buttons.

o	Displays cargo holds, mission cargo, profit columns.

### 	KeyDown()

o	Handles keyboard navigation and buying/selling.

### 	Click()

o	Processes mouse input to buy, sell, or select commodities.

### 	Buy()

o	Adjusts player's cargo, credits, trade basis, and profit logs.

## Data Structures

### Purpose	Data Structure

Player Data	PlayerInfo

System Commodity Prices	System

Commodities List	vector<Commodity>

Cargo Holdings	unordered_map<string, int>

Outfits in Cargo	unordered_map<string, Outfit*>

Cargo/Prices/Profits	int64_t

UI Colors/Fonts/Rects	Color, Font, Rectangle

UI Layout Constants

Constant	Purpose

NAME_X	X-position for commodity names

PRICE_X	X-position for prices

LEVEL_X	Price level indicators

PROFIT_X	Profit columns

BUY_X	Buy buttons

SELL_X	Sell buttons

HOLD_X	Held amounts

________________________________________

# 📌 Final Summary

This combined system powers Endless Sky's dynamic, sandbox universe, enabling:

•	Procedural mission generation with branching, condition-based availability.

•	Data-driven ship configuration via modular, hierarchical definitions.

•	Weapons with advanced behaviors, submunitions, effects, and range-based damage.

•	Interactive star map navigation, overlays, and system/planet searches.

•	Commodity trading with real-time market systems, profit tracking, and trade basis adjustments.

This system architecture balances flexibility with performance, allowing mission designers, ship modders, and economy balancers to dynamically shape the game's world and player experience.

