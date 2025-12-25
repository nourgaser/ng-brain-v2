config.set {
    indexPage = "index",
    treeview = {
        position = "lhs",
        width = "250px",
        exclusions = {
            "^_plug",
            "^Library",
            "^CONFIG.md"
        }
    },
    editor = {
        fontFamily = "Fira Code, monospace"
    },
    actionButtons = {
        {
            icon = "sidebar",
            description = "Toggle Tree View",
            run = function()
                editor.invokeCommand("Tree View: Toggle")
            end
        }
    }
}