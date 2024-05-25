import hou
import time
from PySide2 import QtGui, QtWidgets, QtCore

class MainWindow(QtWidgets.QWidget):
    def __init__(self):
        super(MainWindow, self).__init__()
        self.setWindowTitle("Light Switcher V1.0")
        self.setGeometry(100, 100, 400, 500)
                
        self.list_widget = QtWidgets.QListWidget(self)
        self.list_widget.setGeometry(10, 10, 380, 280)

        # Multi Selection
        self.list_widget.setSelectionMode(QtWidgets.QAbstractItemView.ExtendedSelection)
        self.populate_light_list()

        # Group box for adding a prefix
        self.group_box = QtWidgets.QGroupBox("Add Prefix", self)
        self.group_box.setCheckable(True)
        self.group_box.setGeometry(10, 300, 380, 60)

        group_layout = QtWidgets.QHBoxLayout()
        self.group_box.setLayout(group_layout)
        self.name_prefix = QtWidgets.QLineEdit()
        group_layout.addWidget(self.name_prefix)

        # Button Transference
        self.transfer_button = QtWidgets.QPushButton("Convert to Solaris", self)
        self.transfer_button.setGeometry(10, 400, 380, 40)
        self.transfer_button.clicked.connect(self.transfer_to_stage)

        # Progress Bar
        self.progress_bar = QtWidgets.QProgressBar(self)
        self.progress_bar.setGeometry(10, 450, 380, 35)
        self.progress_bar.setValue(0)

    def populate_light_list(self):
        # Use type().name() to get a list of all lights from /obj
        lights = [node for node in hou.node("/obj").children() if node.type().name() in ["envlight", "hlight::2.0"]]
        for light in lights:
            self.list_widget.addItem(light.name())

    def set_common_parameters(self, light, stg_node):
        # Set name
        original_name = light.name()
        if self.group_box.isChecked():
            prefix = self.name_prefix.text()
            new_name = f"{prefix}_{original_name}"
        else:
            new_name = original_name
        stg_node.setName(new_name)

        # Set intensity
        intensity = light.parm("light_intensity").eval()
        stg_node.parm("xn__inputsintensity_i0a").set(intensity)

        # Set exposure
        exposure = light.parm("light_exposure").eval()
        stg_node.parm("xn__inputsexposure_vya").set(exposure)

        # Set color
        color = light.parmTuple("light_color").eval()
        stg_node.parmTuple("xn__inputscolor_zta").set(color)

        # Set translation
        translate = light.parmTuple("t").eval()
        stg_node.parmTuple("t").set(translate)

        # Set rotation
        rotate = light.parmTuple("r").eval()
        stg_node.parmTuple("r").set(rotate)

    def lights_names_exist(self, light_names):
        stage = hou.node("/stage")
        existing_nodes = stage.children()
        existing_names = [node.name() for node in existing_nodes]
        existing_lights = [name for name in light_names if name in existing_names]

        if existing_lights:
            message = "Light(s) has been already converted:\n" + "\n".join(existing_lights)
            QtWidgets.QMessageBox.information(self, "Lights Exist", message)
            return True
        return False

    def update_progress_bar(self):
        self.progress_bar.setValue(0)
        for i in range(101):
            self.progress_bar.setValue(i)
            time.sleep(0.01)  
            QtWidgets.QApplication.processEvents()  

    def transfer_to_stage(self):
        selected_items = self.list_widget.selectedItems()
        stage = hou.node("/stage")

        # Get the name of the nodes in /obj
        selected_names = [item.text() for item in selected_items]
        selected_nodes = [hou.node("/obj/" + name) for name in selected_names]

        # Split lights by type
        env_lights = [node for node in selected_nodes if node.type().name() == "envlight"]
        h_lights = [node for node in selected_nodes if node.type().name() == "hlight::2.0"]

        # Check if the names already exist in /stage
        if self.lights_names_exist(selected_names):
            return

        # Transfer to Solaris
        if h_lights:
            for light in h_lights:
                light_type_value = light.parm("light_type").eval()             
                spot_light_type_value = light.parm("coneenable").eval()

                # POINT LIGHT
                if light_type_value == 0 and spot_light_type_value == 0:
                    h_light_stg = stage.createNode("light::2.0")
                    self.set_common_parameters(light, h_light_stg)

                # SPOTLIGHT
                elif light_type_value == 0 and spot_light_type_value == 1:
                    h_light_stg = stage.createNode("light::2.0")
                    h_light_stg.parm("lighttype").set(2)  # Set to spotlight
                    h_light_stg.parm("spotlightenable").set(True)
                    cone_angle = light.parm("coneangle").eval()
                    h_light_stg.parm("xn__inputsshapingconeangle_wcbhe").set(cone_angle)
                    self.set_common_parameters(light, h_light_stg)

                # AREA LIGHT
                elif light_type_value == 2:
                    h_light_stg = stage.createNode("light::2.0")
                    h_light_stg.parm("lighttype").set(4)  # Set to area light
                    area_size = light.parmTuple("areasize").eval()
                    h_light_stg.parm("xn__inputswidth_zta").set(area_size[0])
                    h_light_stg.parm("xn__inputsheight_mva").set(area_size[1])
                    self.set_common_parameters(light, h_light_stg)

                # DISTANT LIGHT
                elif light_type_value == 7:
                    h_light_stg = stage.createNode("distantlight::2.0")
                    self.set_common_parameters(light, h_light_stg)

        if env_lights:
            for light in env_lights:
                dome_light = stage.createNode("domelight::2.0")
                env_texture = light.parm("env_map").eval()
                dome_light.parm("xn__inputstexturefile_r3ah").set(env_texture)
                self.set_common_parameters(light, dome_light)

        # Update Progress Bar
        self.update_progress_bar()

        # Show Message
        QtWidgets.QMessageBox.information(self, "Transfer Lights", "The Conversion has been successful")
        
        # Close Window 
        self.close()

window = MainWindow()
window.show()
