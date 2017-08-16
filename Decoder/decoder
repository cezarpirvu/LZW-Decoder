import os, sys


# LZW decompressor - author Cezar-Costin Pirvu

# class for handling the decoder
class Decoder:
    # class constructor
    def __init__(self, file_path, dictionary_size):
        self.file_path = file_path
        self.dictionary = self.default_dictionary()
        self.initial_dictionary = dict(self.dictionary)
        self.dictionary_size = pow(2, dictionary_size)

    # function used for creating the initial dictionary
    def default_dictionary(self):
        # construct the initial dictionary 0 -> "\x00", 1 -> "\x01" ... 65 -> "A", 66 -> "B"
        dictionary = dict()
        for index in range(0, 256):
            dictionary.update({index:chr(index)})
        return dictionary


    # generator function to have access to the index, previous, current and next bytes in the file
    def current_next_items(self, iterable):
        previous_item = None
        index = 0
        iterator = iter(iterable)
        current_item = next(iterator)
        for next_item in iterator:
            yield (index, previous_item, current_item, next_item)
            index += 1
            previous_item = current_item
            current_item = next_item
        yield (index, previous_item, current_item, None)


    # function that reads the given compressed file and extracts the codes
    def extract_codes(self, file_path):
        # check to see if the file exists
        try:
            # open the file and read each byte
            with open(file_path, 'rb') as f:
                # size of the input file
                file_size = os.path.getsize(file_path)
                # codes list
                codes = []
                for index, previous, current, next in self.current_next_items(f.read()):
                    # check if only one code is in the file (if the size of the file has 2 bytes padded to 16-bits)
                    if file_size == 2:
                        byte = bin(current)[2:].rjust(8, '0') + bin(next)[2:].rjust(8, '0')
                        # convert the codes from bin to decimal and add them to the list of codes
                        codes.append(int(byte.encode(), 2))
                    # for the rest of the file
                    if next is not None:
                        if index % 3 == 0:
                            byte = bin(current)[2:].rjust(8, '0') + bin(next)[2:].rjust(8, '0')[:4]
                            codes.append(int(byte.encode(), 2))
                        elif index % 3 == 1:
                            byte = bin(current)[2:].rjust(8, '0')[-4:] + bin(next)[2:].rjust(8, '0')
                            codes.append(int(byte.encode(), 2))
                    else:
                        # check if there is an odd number of codes
                        if file_size % 3 == 1:
                            codes.pop()
                            byte = bin(previous)[2:].rjust(8, '0') + bin(current)[2:].rjust(8, '0')
                            codes.append(int(byte.encode(), 2))
            return codes
        except FileNotFoundError:
            print("File not found!")
            exit(1)


    # function used for decompressing a file
    def decompress(self):
        dictionary_size = self.dictionary_size
        # dictionary size must be between 256 and 4096
        if dictionary_size > 4096 or dictionary_size < 256:
            print("Dictionary size must be between 256 and 4096 (8 or 12 bits)")
            exit(1)
        # reconstructed output
        output = ''
        # store the previous code read
        prev_code = ''
        # extract the codes from the file
        codes = self.extract_codes(self.file_path)

        # get the first code from the list
        if codes[0] in self.dictionary.keys():
            output += self.dictionary.get(codes[0])
            prev_code = codes[0]

        # now for the rest
        for code in codes[1:]:
            # current dictionary size
            dict_size = len(self.dictionary)
            # previous entry in the dictionary
            prev_entry = self.dictionary.get(prev_code)
            # current entry in the dictionary
            curr_entry = self.dictionary.get(code)

            if code in self.dictionary.keys():
                # update and store the decoded message
                output += curr_entry
                # if current dictionary size is larger than 4096, reset the dictionary
                if dict_size >= dictionary_size:
                    self.dictionary = dict(self.initial_dictionary)
                # insert the new entry in the dictionary
                self.dictionary.update({dict_size: prev_entry + curr_entry[0]})
                prev_code = code
            else:
                # update and store the decoded message
                output += prev_entry + prev_entry[0]
                # if current dictionary size is larger than 4096, reset the dictionary
                if dict_size >= dictionary_size:
                    self.dictionary = dict(self.initial_dictionary)
                # insert the new entry in the dictionary
                self.dictionary.update({dict_size: prev_entry + prev_entry[0]})
                prev_code = dict_size
        return output

print ('Usage: decoder <input_file_name> <output_file_name>')
try:
    # try to open the file for writing
    with open(sys.argv[2], 'w') as f:
        # call the decoder with with the dictionary size of a maximum of 12 bits
        decoder = Decoder(sys.argv[1], 12)
        # write to file
        f.write(str(decoder.decompress().encode('utf-8')))
except:
    print('An error occured')
    exit(1)
print ('Success!')

