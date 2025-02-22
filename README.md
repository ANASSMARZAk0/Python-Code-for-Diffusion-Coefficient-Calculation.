# Python-Code-for-Diffusion-Coefficient-Calculation.
import numpy as np
import ipywidgets as widgets
from IPython.display import display

# Function to calculate D_AB using the Hsu and Chen method
def calculate_d_ab_hsu_chen(x_A, x_B, D_AB_circ, D_BA_circ, a_BA, a_AB, r_A, r_B, q_A, q_B, lambda_A, lambda_B, T):
    try:
        # Calculate theta_j (Eq. 11-10.9)
        theta_A = (x_A * q_A) / (x_A * q_A + x_B * q_B)
        theta_B = (x_B * q_B) / (x_A * q_A + x_B * q_B)

        # Calculate phi_i (Eq. 11-10.11)
        phi_A = (x_A * lambda_A) / (x_A * lambda_A + x_B * lambda_B)
        phi_B = (x_B * lambda_B) / (x_A * lambda_A + x_B * lambda_B)

        # Calculate tau_ji (Eq. 11-10.10)
        tau_BA = np.exp(-a_BA / T)
        tau_AB = np.exp(-a_AB / T)

        # Calculate theta_ji (Eq. 11-10.8)
        theta_BA = (theta_B * tau_BA) / (theta_A * tau_AB + theta_B * tau_BA)
        theta_AB = (theta_A * tau_AB) / (theta_A * tau_AB + theta_B * tau_BA)
        theta_AA = (theta_A * tau_AB) / (theta_A * tau_AB + theta_B * tau_BA)
        theta_BB = (theta_B * tau_BA) / (theta_A * tau_AB + theta_B * tau_BA)

        # Calculate ln(D_AB) using Eq. (11-10.7)
        term1 = x_B * np.log(D_AB_circ) + x_A * np.log(D_BA_circ)
        term2 = 2 * (x_A * np.log(x_A / phi_A) + x_B * np.log(x_B / phi_B))
        term3 = 2 * x_A * x_B * (
            (phi_A / x_A) * (1 - lambda_A / lambda_B) +
            (phi_B / x_B) * (1 - lambda_B / lambda_A)
        )
        term4 = x_B * q_A * (
            (1 - theta_BA**2) * np.log(tau_BA) +
            (1 - theta_BB**2) * tau_AB * np.log(tau_AB)
        )
        term5 = x_A * q_B * (
            (1 - theta_AB**2) * np.log(tau_AB) +
            (1 - theta_AA**2) * tau_BA * np.log(tau_BA)
        )

        ln_D_AB = term1 + term2 + term3 + term4 + term5
        D_AB = np.exp(ln_D_AB)

        return D_AB
    except Exception as e:
        return f"Error: {str(e)}"

# Create input widgets with empty initial values
x_A_input = widgets.FloatText(description='x_A:')
x_B_input = widgets.FloatText(description='x_B:')
D_AB_circ_input = widgets.FloatText(description='D_AB^o (cm^2/s):')
D_BA_circ_input = widgets.FloatText(description='D_BA^o (cm^2/s):')
a_BA_input = widgets.FloatText(description='a_BA:')
a_AB_input = widgets.FloatText(description='a_AB:')
r_A_input = widgets.FloatText(description='r_A:')
r_B_input = widgets.FloatText(description='r_B:')
q_A_input = widgets.FloatText(description='q_A:')
q_B_input = widgets.FloatText(description='q_B:')
lambda_A_input = widgets.FloatText(description='lambda_A:')
lambda_B_input = widgets.FloatText(description='lambda_B:')
T_input = widgets.FloatText(description='Temperature (K):')
D_AB_exp_input = widgets.FloatText(description='Experimental D_AB (cm^2/s):')

# Create calculate button
calculate_button = widgets.Button(description="Calculate D_AB and Error")

# Output widget to display results
output = widgets.Output()

# Function to handle button click
def on_button_click(b):
    try:
        # Get values from input widgets
        x_A = x_A_input.value
        x_B = x_B_input.value
        D_AB_circ = D_AB_circ_input.value
        D_BA_circ = D_BA_circ_input.value
        a_BA = a_BA_input.value
        a_AB = a_AB_input.value
        r_A = r_A_input.value
        r_B = r_B_input.value
        q_A = q_A_input.value
        q_B = q_B_input.value
        lambda_A = lambda_A_input.value
        lambda_B = lambda_B_input.value
        T = T_input.value
        D_AB_exp = D_AB_exp_input.value

        # Calculate D_AB
        D_AB = calculate_d_ab_hsu_chen(x_A, x_B, D_AB_circ, D_BA_circ, a_BA, a_AB, r_A, r_B, q_A, q_B, lambda_A, lambda_B, T)

        # Calculate error
        error = ((D_AB - D_AB_exp) / D_AB_exp) * 100

        # Display the result
        with output:
            output.clear_output()
            print(f"Calculated D_AB: {D_AB:.4e} cm^2/s")
            print(f"Experimental D_AB: {D_AB_exp:.4e} cm^2/s")
            print(f"Error: {error:.2f}%")
    except ValueError:
        with output:
            output.clear_output()
            print("Error: Please enter valid numbers in all fields.")

# Attach the function to the button
calculate_button.on_click(on_button_click)

# Display all widgets in a vertical layout
input_widgets = widgets.VBox([
    x_A_input, x_B_input, D_AB_circ_input, D_BA_circ_input, a_BA_input, a_AB_input,
    r_A_input, r_B_input, q_A_input, q_B_input, lambda_A_input, lambda_B_input,
    T_input, D_AB_exp_input, calculate_button, output
])

# Display the layout
display(input_widgets)
