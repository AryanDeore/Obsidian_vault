I want to create a repo that can replicate my current macs setup in a new mac. I originally had a repo called mac_setup: `https://github.com/AryanDeore/Mac_setup` but that repo didn’t have an easy way to replicate the mac system settings. Then I found a repo called dot files that does this: `https://github.com/kunchenguid/dotfiles/tree/main`
This repo also adds a step to replicate agents, skills, harness etc.
I want to 

1. Install Determinate-Nix: `curl --proto '=https' --tlsv1.2 -sSf -L https://install.determinate.systems/nix | sh -s -- install`
2. After installation, the setup says to run this command: ![[Screenshot 2026-09-16 at 11.50.59 PM.png]]
3. 

Add pure preset for starship: https://starship.rs/presets/pure-preset

6. [Agents.md](https://github.com/jasonku09/agents-md-snippets) by Jason Ku
	1. Global Agents.md: 
		1. When doing bug fixes, always start with reproducing the bug in an end-to-end setting as closely aligned with how an end user would experience it as possible. this makes sure you find the real problem, so your fix will actually solve it
7. **Verify**:  [No-mistake](https://github.com/kunchenguid/no-mistakes) by Kun Chen

Grill-me-with-ui or lavish:
- Lavish for UI
- Grill-me-with-ui for architecting


AXI.
**Automic vault** for secrets management
**/firstmate** for multi agent orchestration
**OpenTofu** for Iaac
[OpenAI server compaction for pi](https://github.com/algal/pi-openai-server-compaction)
**Herdr** as terminal


### MacOS defaults:
##### Dock
- Dock autohide: `defaults write com.apple.dock "autohide" -bool "true" && killall Dock`
- Reduce the dock speed to 0: `defaults write com.apple.dock "autohide-time-modifier" -float "0" && killall Dock`
- Doc position to right: `defaults write com.apple.dock "orientation" -string "right" && killall Dock`
- Dock icon size: `defaults write com.apple.dock "tilesize" -int "36" && killall Dock`
- Autohide delay: `defaults write com.apple.dock "autohide-delay" -float "0" && killall Dock`
- No show recent: `defaults write com.apple.dock "show-recents" -bool "false" && killall Dock`
- Minimization- scale: `defaults write com.apple.dock "mineffect" -string "scale" && killall Dock`

- Screenshot location: `defaults write com.apple.screencapture "location" -string "~/Pictures" && killall SystemUIServer`
##### Finder:
- Show file extensions: `defaults write NSGlobalDomain "AppleShowAllExtensions" -bool "true" && killall Finder`
- Show hidden files: `defaults write com.apple.finder "AppleShowAllFiles" -bool "true" && killall Finder`
- Show path bar: `defaults write com.apple.finder "ShowPathbar" -bool "true" && killall Finder`
- View style- columns: `defaults write com.apple.finder "FXPreferredViewStyle" -string "clmv" && killall Finder`
- Folders on top: `defaults write com.apple.finder "_FXSortFoldersFirst" -bool "true" && killall Finder`
- Column view: `defaults write com.apple.finder "FXPreferredViewStyle" -string "clmv" && killall Finder`

##### Mouse:
- Disable mouse accel: `defaults write NSGlobalDomain com.apple.mouse.linear -bool "true"`
- Mouse speed: `defaults write NSGlobalDomain com.apple.mouse.scaling -float "0.875"`


- Trackpad tap to click
- Finder create desktop
- Preview - pdf continuous scroll
- Scroll wheel direction
- Dark mode 
- 