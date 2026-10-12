# Project Folder Structure

## 1. What a Fresh Project Gives You

**React Native CLI (bare):**

```
my-app/
    android/          # native Android project
    ios/              # native iOS project
    App.tsx           # root component
    index.js          # entry point: registers App
    package.json
```

Put your own code in a `src/` folder (a common convention) to keep it separate from config files.

## 2. Approach A: Organize by Feature (Domain-Based)

```
src/
    features/
        plants/
            components/
                PlantCard.tsx
                PlantForm.tsx
            screens/
                PlantListScreen.tsx
                PlantDetailsScreen.tsx
            hooks/
                usePlants.ts
            services/
                plantsApi.ts
            types.ts
            index.ts          # public exports of the feature
        watering/
            components/
            hooks/
            services/
            types.ts
            index.ts
        auth/
            ...
    shared/                   # used by 2+ features
        components/
            Button.tsx
            Modal.tsx
        hooks/
        utils/
        theme/
    navigation/
    app/                      # app-level setup (providers, root component)
```

**Pros:**

- Everything for a feature lives together, so changes are local.
- It scales well with large apps and teams, since people can work on different features without conflicts.
- Deleting or extracting a feature is easy.

**Cons:**

- You have to decide what is "shared" and what belongs to a feature, and that line moves over time.
- It is more folders and more ceremony for tiny apps.
- Cross-feature imports can get tangled if you don't enforce boundaries (features importing each other's internals).

## 3. Approach B: Organize by Component

```
src/
    components/                     # shared, reusable components
        Button/
            Button.tsx
            Button.styles.ts
            Button.types.ts
            Button.test.tsx
            index.ts
        PlantCard/
            PlantCard.tsx
            PlantCard.styles.ts
            PlantCard.types.ts
            PlantCard.test.tsx
            index.ts
            components/             # sub-components used ONLY by PlantCard
                WaterBadge/
                    WaterBadge.tsx
                    WaterBadge.styles.ts
                    index.ts
            hooks/                  # logic used ONLY by PlantCard
                usePlantCard.ts
    screens/                        # each screen is also a component folder
        HomeScreen/
            HomeScreen.tsx
            HomeScreen.styles.ts
            index.ts
            components/             # components used ONLY by this screen
                PlantList/
                    PlantList.tsx
                    index.ts
        SettingsScreen/
            ...
    hooks/                          # truly global hooks
    services/                       # API, storage (not tied to a component)
    theme/                          # colors, spacing
    navigation/
```

**Pros:**

- **Everything about a component is in one place.** Editing, moving, or deleting a component means touching one folder.
- **Easy to find things.** The folder name is the component name.
- **Private pieces stay private.** Sub-components and hooks don't pollute a global folder.
- **Good for design systems and UI libraries.** Each component is self-contained and portable.
- **Scales better than by-type for the UI layer**, with no giant components.
- **Tests and styles are co-located**, so you don't forget to update them.

**Cons:**

- **Lots of folders and tiny files.** A simple Button becomes four or five files, which is heavy for small components.
- **Editor tab confusion.** Many `index.ts` files all show the same tab name.
- **Cross-component features get scattered.** The "plants" feature is spread across PlantCard, PlantForm, PlantListScreen, and so on. By-feature groups these together.
- **The promote-when-reused rule needs discipline.** Without it, deep nesting and duplicated components appear.
- **Barrel files** (`index.ts`) can slow bundling and cause circular imports in large projects.

## 4. Approach C: Big Component Owns Its Sub-Components

```
src/
    components/
        Button/                          # small shared component
            Button.tsx
            index.ts
    screens/
        PlantDashboard/                  # the BIG component
            PlantDashboard.tsx           # root: owns state, composes children
            PlantDashboard.styles.ts
            PlantDashboard.types.ts      # types shared by its sub-components
            usePlantDashboard.ts         # logic shared by its sub-components
            index.ts                     # ONLY export: PlantDashboard
            components/
                DashboardHeader/
                    DashboardHeader.tsx
                    DashboardHeader.styles.ts
                    index.ts
                PlantList/
                    PlantList.tsx
                    PlantList.styles.ts
                    index.ts
                    components/          # sub-component of a sub-component
                        PlantRow/
                            PlantRow.tsx
                            PlantRow.styles.ts
                            index.ts
                            components/
                                WaterButton/
                                    WaterButton.tsx
                                    index.ts
                EmptyState/
                    EmptyState.tsx
                    index.ts
```

**Pros:**

- **High cohesion.** A big component and all its private pieces are one unit. Moving or deleting the dashboard means moving or deleting one folder.
- **The structure documents the UI.** The folder tree shows how the component is composed.
- **No accidental reuse.** `PlantRow` doesn't appear in a shared list where someone might wrongly reuse it.
- **Clear ownership.** It is obvious which code belongs to which part of the UI.
- **Easy extraction.** If a big component grows into its own package or feature, you lift the folder as is.

**Cons:**

- **Deep nesting.** Long paths, long relative imports (`../../../`), and a lot of clicking through folders to reach a small component.
- **Many same-named files in editor tabs.** Dozens of `index.ts` files are hard to tell apart.
- **Prop drilling or context overhead.** Deep trees tempt you into passing props through many layers.
