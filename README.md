# Civil-design-program-
This code can be used for concrete and beam designing 
Civil Engineering Design Calculator - British Standards
BS 8110, BS 5950, BS 6399

import math

class ConcreteDesign:
    """Concrete design according to BS 8110"""
    
    def __init__(self, fcu=30, fy=500):
        self.fcu = fcu  # Concrete strength MPa
        self.fy = fy    # Steel strength MPa
        self.gamma_c = 1.5  # Concrete partial factor
        self.gamma_s = 1.15  # Steel partial factor
    
    def beam_moment_capacity(self, b, d, As):
        """Calculate moment capacity of rectangular beam"""
        x = (0.87 * self.fy * As) / (0.45 * self.fcu * b)
        z = d - 0.45 * x
        return min(0.156 * self.fcu * b * d**2 / 1e6, 
                  0.87 * self.fy * As * z / 1e6)
    
    def required_reinforcement(self, M, b, d):
        """Calculate required reinforcement area"""
        K = M * 1e6 / (b * d**2 * self.fcu)
        if K > 0.156:
            raise ValueError("Compression reinforcement required")
        
        z = d * (0.5 + math.sqrt(0.25 - K/0.9))
        return M * 1e6 / (0.87 * self.fy * z)

class SteelDesign:
    """Steel design according to BS 5950"""
    
    def __init__(self, py=275):
        self.py = py  # Steel design strength
    
    def universal_beam_capacity(self, section):
        """Calculate capacity for common UB sections"""
        capacities = {
            "UB 203x133x25": {"Moment": 70, "Shear": 200},
            "UB 254x146x31": {"Moment": 110, "Shear": 300},
            "UB 305x165x40": {"Moment": 180, "Shear": 450}
        }
        return capacities.get(section, {"Moment": 0, "Shear": 0})

class Loading:
    """Loading calculations according to BS 6399"""
    
    @staticmethod
    def combination(dead_load, live_load, load_type="ultimate"):
        """Load combinations"""
        factors = {
            "ultimate": (1.4, 1.6),
            "serviceability": (1.0, 1.0)
        }
        gamma_g, gamma_q = factors[load_type]
        return (dead_load * gamma_g + live_load * gamma_q)

def calculate_deflection(load, span, EI, load_type="point"):
    """Calculate beam deflection"""
    if load_type == "point":
        return (load * 1000) * (span * 1000)**3 / (48 * EI)
    else:  # UDL
        return 5 * (load * 1000) * (span * 1000)**4 / (384 * EI)
