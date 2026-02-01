library IEEE; use IEEE.STD_LOGIC_1164.all; use IEEE.NUMERIC_STD.all;

entity main is
port(
	clk, data_in, rst_n : in STD_LOGIC;
	stored_val : out STD_LOGIC_VECTOR(7 downto 0);
	timer_out : out UNSIGNED(3 downto 0)

);
end entity main; 

architecture synth of main is
--signal declaration
CONSTANT clk_freq : integer := 25_000_000; --ENTER FREQUENCY HERE!
CONSTANT baud_rate : integer := 9600; --ENTER BAUDRATE HERE!
CONSTANT bit_time : integer := clk_freq/baud_rate;
signal reg : STD_LOGIC_VECTOR(7 downto 0) := (others=>'0');
signal counter : UNSIGNED(3 downto 0);
signal timer : UNSIGNED(20 downto 0);
type statetype is (waiting, reading);
signal state : statetype;
type sysstate is (IDLE, READING, STOP);
signal sys_state : sysstate := IDLE; 
begin
--main control process
process(clk)
begin
if(rising_edge(clk)) then
	--reset conditions
	if(rst_n = '0') then
		sys_state <= IDLE;
		state <= reading;
		counter <= (others => '0');
	else
	--initialization
	if(sys_state = IDLE and state = reading and data_in = '0') then
		state <= waiting;
		counter <= (others => '0');
	elsif(sys_state = IDLE and state = waiting) then
	--initial waiting (1.5 times)
	if(timer = bit_time + bit_time/2) then
		state <= reading;
		sys_state <= READING;
	end if; --end initial waiting
	end if; --end initialization if
	--reading normal data
	if(sys_state = READING and state = reading) then
		state <= waiting;
		counter <= counter + 1;
	end if;
	--waiting between normal data inputs
	if(sys_state = READING and state = waiting) then
		if(timer = bit_time and counter /= 8) then		
		state <= reading;
		--ending condition, triggers after the last bit has been inputted
		elsif(counter = 8 and state = waiting) then
			sys_state <= STOP;
		end if;
	end if;
	--stop condition
	if(sys_state = STOP) then
		if(timer = bit_time) then
			state <= reading;
			sys_state <= IDLE;
		end if;
	end if;
	
	end if;
end if;
end process;

--timer process
process(clk) 
begin
if(rising_edge(clk)) then
if(rst_n='1') then
	if(state = waiting) then
		timer <= timer + 1;
	else
		timer <= (others => '0');
	end if;
else
	timer <= (others => '0');
end if;
end if;
end process;

--reader process
process(clk)
begin
if(rising_edge(clk)) then
	if(state = reading and sys_state = READING) then
		reg(TO_INTEGER(counter)) <= data_in; 
	end if;
	if(state = reading and sys_state = IDLE) then
		reg <= (others=>'0');
	end if;
end if;
end process;

--outputs
stored_val <= reg;
timer_out <= counter;
end architecture synth;